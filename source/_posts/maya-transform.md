title: 基于Maya批量制作带有Transform动画的usdz技术探索
date: 2019-12-26
author: 西狂
subtitle: 本篇blog主要是探索批量制作带有Transform动画的usdz文件。Transform动画部分使用Maya软件进行调研生成的
tags: [Maya, Transform]

categories: Maya
---
## 简介
&emsp;&emsp;本篇blog主要是探索批量制作带有Transform动画的usdz文件。Transform动画部分使用Maya软件进行调研生成的。

## 主要技术流程
 1. 模型及材质准备
 2. 动画制作
 3. usdz制作
 #### 1.模型及材质准备
 &emsp;&emsp;首先建模。这里注意模型及材质命名要规范，不然在制作动画时，导入三维动画软件时，名称会被改，如“：”，很多时候会被“-”代替。
 &emsp;&emsp;然后制作pbr材质，因为usdz是实时渲染，不能够使用离线渲染材质。图片尺寸不能超过2048，且所有图片都要是2^N^次方。在测试usdz生成时，使用数值如color，无法正常显示，但是用color_map能正常显示。因此各个通道的材质都一图片形式表示。通道包括 color_map normal_map roughness_map metallic_map emissive_map ,但材质未用到相应的通道，可以不用生成。
 ![kujiale](https://img-blog.csdnimg.cn/20191220012617688.png)
 &emsp;&emsp;最后制作mtl文件，该文件主要是用于描述模型及材质对应关系的，为usdz生成时，材质贴图能够对应到模型，并打包进usdz。可以是如下形式：
```c
newmtl sofa_leg
	map_Kd sofa_leg_color.jpg
	map_Roughness sofa_leg_roughness.png
	map_Metallic sofa_leg_metallic.png
```
#### 2.动画制作
&emsp;&emsp;主要是以旋转为例，进行脚本编写。输入是模型.obj文件，输出为.abc文件。执行过程中，不用导入材质。而obj导入进三维动画软件中，材质丢失或用默认材质，总之材质是有问题的，但是导出的abc文件，group 名称是没有变的，因此在生成usdz时，用group名称，来对应材质。
&emsp;&emsp;下面是Maya动画中Python脚本文件：
```c
#maya_anim.py

import maya.standalone
import maya.cmds as cmds
import os

# selectionList:选择要做动画的物体
# Axis：坐标轴，如 'rotateY',具体参考Maya官方文档
# startTime：动画开始时间
# endTime:动画结束时间
# startValue：起始数值，如旋转从0°开始
# endValue：结束数值，如旋转到180°。
def Anim_SingleTransform(selectionList, Axis, startTime, endTime, startValue, endValue):
    if len(selectionList) >= 1:
        for objectName in selectionList:
            cmds.cutKey(objectName, time=(startTime, endTime), attribute=Axis)
            cmds.setKeyframe(objectName, time=startTime, attribute=Axis, value=startValue)
            cmds.setKeyframe(objectName, time=endTime, attribute=Axis, value=endValue)
            cmds.selectKey(objectName, time=(startTime, endTime), attribute=Axis, keyframe=True)
            cmds.keyTangent(inTangentType='linear', outTangentType='linear')

#objPath: 需要输入obj文件的路径
#ABCPath：输出abc动画文件路径
def Process(objPath, abcPath):
    if os.path.exists(objPath):
        maya.standalone.initialize()
        try:
            cmds.loadPlugin('AbcExport.mll')
        except RuntimeError:
            print('load plugin faild')
        cmds.playbackOptions(minTime='0sec', maxTime='0.20833333sec')
        cmds.file(objPath, i=True)
        cmds.select(all=True)
        startTime = cmds.playbackOptions(query=True, minTime=True)
        endTime = cmds.playbackOptions(query=True, maxTime=True)
        selectionList = cmds.ls(selection=True, type='transform')
        Anim_SingleTransform(selectionList, 'rotateY', startTime, endTime, 0, 180)
        cmds.AbcExport(j = '-frameRange 1 6 -uvWrite -dataFormat ogawa -file ' + abcPath)
        maya.standalone.uninitialize()
    else:
        print('-------------obj file does not exist----------')
# objPath, abcPath 由批量生产abc文件的Python脚本传入（main.py）
objPath = ""
abcPath = ""
Process(objPath, abcPath)
```
&emsp;&emsp;下面是批量生产abc文件Python脚本文件：

```c
# main.py

import os
import subprocess

cwd = os.path.dirname(__file__)
mayaLocation = r'.\your maya installed location\Autodesk\Maya2017\bin'
mayapyPath = mayaLocation + r'\mayapy.exe'
mayaPath = mayaLocation + r'\maya.exe'
mayaScript = os.path.join(cwd, 'maya_anim.py')

def abc():
    cmd = "\"" + mayapyPath + "\"" + " " + "\"" + mayaScript + "\""
    sub = subprocess.Popen(cmd, shell= True)
    sub.wait()

# 修改maya_anim.py中文件输入和输出路径，当然也可以用commandline方式。
def changeScriptValue(inPath, outPath):
    inPath = '\"' + inPath.replace('\\','/') + '\"\n'
    outPath = '\"' + outPath.replace('\\', '/') + '\"\n'
    file_data = ""
    with open(mayaScript, 'r') as f1:
        for line in f1:
            if line.startswith('objPath = '):
                line = 'objPath = ' + inPath
            if line.startswith('abcPath = '):
                line = 'abcPath = ' + outPath
            file_data += line
    with open(mayaScript, 'w') as f2:
        f2.write(file_data)

if __name__ == '__main__':
    # 可以循环设置参数，进行批量生产
    abcInpath = ''
    abcOutPath = ''
    changeScriptValue(abcInpath,abcOutPath)
    abc()

```

#### 3.usdz制作
&emsp;&emsp;根据abc文件和材质贴图，以及对应关系描述文件mtl，进行usdz文件生成。
&emsp;&emsp;转换脚本如下：
```c
#abc2usdz.py

import os
import sys
import subprocess

#获得模型group名称与材质对应关系，需要从obj文件中获得。
def get_mesh_mat(assertPath):
    dic_mesh_mat = {}
    f = open(assertPath)
    lines = f.readlines()
    f.close()
    g = ''
    for line in lines:
        line = line.strip('\r\n')
        if line.startswith('g '):
            g = line.split(' ')[1]
        if line.startswith('usemtl '):
            m = line.split(' ')[1]
            if g != '' and m != '':
                dic_mesh_mat[m] = g
    return dic_mesh_mat

# inputPath：文件输入路径，该路径下需要包含obj文件，abc文件，mtl文件，材质贴图。
# outputPath：usdz存储位置。
def generate_usdz(inputPath,outputPath):
    if os.path.exists(inputPath + "models_baked.mtl"):
        fmtl = open(inputPath + "models_baked.mtl")
        cmd = 'xcrun usdz_converter ' + inputPath + 'models_baked.abc ' + outputPath + 'models_baked.usdz'
        assertPath = os.path.join(inputPath, 'models_baked.obj')
        dic_mat_mesh = get_mesh_mat(assertPath)
        for line in fmtl:
            if len(line) > 2 :
                if(len(line.split(' ')) == 2):
                    v0, v1 = line.split(' ')
                    if v0 == 'newmtl' :
                        cmd += ' -g ' + dic_mat_mesh[v1[0 : len(v1)-1]]
                    if v0 == '\tmap_Kd' :
                        cmd += ' -color_map ' + outputPath + v1[0 : len(v1)-1]
                    if v0 == '\tmap_Normal' :
                        cmd += ' -normal_map ' + outputPath + v1[0 : len(v1)-1]
                    if v0 == '\tmap_Roughness' :
                        cmd += ' -roughness_map ' + outputPath + v1[0 : len(v1)-1]
                    if v0 == '\tmap_Metallic' :
                        cmd += ' -metallic_map ' + outputPath + v1[0 : len(v1)-1]
                    if v0 == '\tmap_Emissive' :
                        cmd += ' -emissive_map ' + outputPath + v1[0 : len(v1)-1]
            line.strip()
        fmtl.close()
    else:
        print("Error: " + subFolder)
    subprocess.call(cmd, shell = True)

# 该脚本执行需要在macOS上运行，且Xcode支持usdz_conveter功能。
if __name__ == '__main__':
     # 可以循环设置参数，进行批量生产
    inputPath = ''
    outputPath = ''
    generate_usdz(inputPath,outputPath)
```
## 动画展示
![kujiale](https://img-blog.csdnimg.cn/2019122003054973.gif)
