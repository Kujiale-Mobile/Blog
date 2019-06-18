title: React Native Modal 的封装与使用
date: 2019-06-18
author: 林河
subtitle: 虽然 RN 自带了一个 Modal 控件，但是在使用过程中它有一些不太好的体验和问题，本篇教你如何解决
tags: [React-Native, 组件]

categories: React-Native
---

# 背景
在使用 React Native（以下简称 RN ，使用版本为 0.59.5） 开发 App 的过程中，有许许多多使用到弹窗控件的场景，虽然 RN 自带了一个 Modal 控件，但是在使用过程中它有一些不太好的体验和问题。
- Android 端的 Modal 控件无法全屏，也就是内容无法从状态栏处开始布局。
- ios 端的 Modal 控件的层级太高，是基于 window 的，如果在 Modal 中打开一个新的 ViewController 界面的时候，将会被 Modal 控件给覆盖住，同时 ios 的 Modal 控件只能弹出一个。

针对上面所发现的问题，我们需要对 RN 的 Modal 控件整体做一个修改和封装，以便于在使用中可以应对各种不同样的业务场景。
# Android FullScreenModal 的封装使用
针对第一个问题，查看了 RN Modal 组件在 Android 端的实现，发现它是对 Android Dialog 组件的一个封装调用，那么假如我能实现一个全屏展示的 Dialog，那么是不是在 RN 上也就可以实现全屏弹窗了。
## Android 原生实现全屏 Dialog
FullScreenDialog 主要实现代码如下
```
public class FullScreenDialog extends Dialog {

    private boolean isDarkMode;
    private View rootView;

    public void setDarkMode(boolean isDarkMode) {
        this.isDarkMode = isDarkMode;
    }

    public FullScreenDialog(@NonNull Context context, @StyleRes int themeResId) {
        super(context, themeResId);
    }

    @Override
    public void setContentView(@NonNull View view) {
        super.setContentView(view);
        this.rootView = view;
    }

    @Override
    public void show() {
        super.show();
        StatusBarUtil.setTransparent(getWindow());
        if (isDarkMode) {
            StatusBarUtil.setDarkMode(getWindow());
        } else {
            StatusBarUtil.setLightMode(getWindow());
        }
        AndroidBug5497Workaround.assistView(rootView, getWindow());
    }
}
```
在这里主要起作用的是 `StatusBarUtil.setTransparent(getWindow());` 方法，它的主要作用是将状态栏背景透明，并且让布局内容可以从 Android 状态栏开始。
```
   /**
     * 使状态栏透明，并且是从状态栏处开始布局
     */
    @TargetApi(Build.VERSION_CODES.KITKAT)
    private static void transparentStatusBar(Window window) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            View decorView = window.getDecorView();
            int option = View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN | View.SYSTEM_UI_FLAG_LAYOUT_STABLE;
            decorView.setSystemUiVisibility(option);
            window.setStatusBarColor(Color.TRANSPARENT);
        } else {
            window.addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS);
        }
    }
```
这里需要注意的是，该方法只有在 Android 4.4 以上才会有效果，不过如今已经是 9012 年了，主流 Android 用户使用的版本应该没有低于 Android 4.4 了吧。
## 封装给 RN 进行相关的调用
### Android 原生部分实现
有了 FullScreenDialog ，下一步就是封装组件给 RN 进行调用了，这里主要的步骤就是参考 RN Modal 的 Android 端实现，然后替换其中的 Dialog 为 FullScreenDialog，最后封装给 RN 进行调用。
```
public class FullScreenModalManager extends ViewGroupManager<FullScreenModalView> {
    @Override
    public String getName() {
        return "RCTFullScreenModalHostView";
    }

    public enum Events {
        ON_SHOW("onFullScreenShow"),
        ON_REQUEST_CLOSE("onFullScreenRequstClose");
        private final String mName;

        Events(final String name) {
            mName = name;
        }

        @Override
        public String toString() {
            return mName;
        }
    }

    @Override
    @Nullable
    public Map getExportedCustomDirectEventTypeConstants() {
        MapBuilder.Builder builder = MapBuilder.builder();
        for (Events event : Events.values()) {
            builder.put(event.toString(), MapBuilder.of("registrationName", event.toString()));
        }
        return builder.build();
    }

    @Override
    protected FullScreenModalView createViewInstance(ThemedReactContext reactContext) {
        final FullScreenModalView view = new FullScreenModalView(reactContext);
        final RCTEventEmitter mEventEmitter = reactContext.getJSModule(RCTEventEmitter.class);
        view.setOnRequestCloseListener(new FullScreenModalView.OnRequestCloseListener() {
            @Override
            public void onRequestClose(DialogInterface dialog) {
                mEventEmitter.receiveEvent(view.getId(), Events.ON_REQUEST_CLOSE.toString(), null);
            }
        });
        view.setOnShowListener(new DialogInterface.OnShowListener() {
            @Override
            public void onShow(DialogInterface dialog) {
                mEventEmitter.receiveEvent(view.getId(), Events.ON_SHOW.toString(), null);
            }
        });
        return view;
    }

    @Override
    public LayoutShadowNode createShadowNodeInstance() {
        return new FullScreenModalHostShadowNode();
    }

    @Override
    public Class<? extends LayoutShadowNode> getShadowNodeClass() {
        return FullScreenModalHostShadowNode.class;
    }

    @Override
    public void onDropViewInstance(FullScreenModalView view) {
        super.onDropViewInstance(view);
        view.onDropInstance();
    }

    @ReactProp(name = "autoKeyboard")
    public void setAutoKeyboard(FullScreenModalView view, boolean autoKeyboard) {
        view.setAutoKeyboard(autoKeyboard);
    }

    @ReactProp(name = "isDarkMode")
    public void setDarkMode(FullScreenModalView view, boolean isDarkMode) {
        view.setDarkMode(isDarkMode);
    }

    @ReactProp(name = "animationType")
    public void setAnimationType(FullScreenModalView view, String animationType) {
        view.setAnimationType(animationType);
    }

    @ReactProp(name = "transparent")
    public void setTransparent(FullScreenModalView view, boolean transparent) {
        view.setTransparent(transparent);
    }

    @ReactProp(name = "hardwareAccelerated")
    public void setHardwareAccelerated(FullScreenModalView view, boolean hardwareAccelerated) {
        view.setHardwareAccelerated(hardwareAccelerated);
    }

    @Override
    protected void onAfterUpdateTransaction(FullScreenModalView view) {
        super.onAfterUpdateTransaction(view);
        view.showOrUpdate();
    }
}
```
在这里有几点需要注意的
- 由于 RN Modal 已经存在了 onShow 和 onRequestClose 回调，这里不能再使用这两个命名，所以这里改成了 onFullScreenShow 和 onFullScreenRequstClose，但是在 js 端还是重新命名成 onShow 和 onRequestClose ，所以在使用过程中还是没有任何变化
- 增加了 isDarkMode 属性，对应上面的状态栏字体的颜色
- 增加了 autoKeyboard 属性，根据该属性判断是否需要自动弹起软件盘

其他的一些属性和用法也就跟 RN Modal 的一样了。
### JS 部分实现
在 JS 部分，我们只需要 Android 的实现就好了，ios 还是沿用原来的 Modal 控件。这里参照 RN Modal 的 JS 端实现如下
```
import React, {Component} from "react";
import {requireNativeComponent, View}  from "react-native";

const FullScreenModal = requireNativeComponent('RCTFullScreenModalHostView', FullScreenModalView)
export default class FullScreenModalView extends Component {

    _shouldSetResponder = () => {
        return true;
    }

    render() {
        if (this.props.visible === false) {
            return null;
        }
        const containerStyles = {
            backgroundColor: this.props.transparent ? 'transparent' : 'white',
        };
        return (
            <FullScreenModal
                style={{position: 'absolute'}}  {...this.props}
                onStartShouldSetResponder={this._shouldSetResponder}
                onFullScreenShow={() => this.props.onShow && this.props.onShow()}
                onFullScreenRequstClose={() => this.props.onRequestClose && this.props.onRequestClose()}>
                <View style={[{position: 'absolute', left: 0, top: 0}, containerStyles]}>
                    {this.props.children}
                </View>
            </FullScreenModal>
        )
    }

}
```
# 使用 RootSiblings 封装 Modal
针对第二个问题，一种方法是通过 ios 原生去封装实现一个 Modal 控件，但是在 RN 的开发过程中，发现了一个第三方库 [react-native-root-siblings](https://github.com/magicismight/react-native-root-siblings) ， 它重写了系统的 AppRegistry.registerComponent 方法，当我们通过这个方法注册根组件的时候，替换根组件为我们自己的实现的包装类。包装类中监听了目标通知 siblings.update，接收到通知就将通知传入的组件视图添加到包装类顶层，然后进行刷新显示。通过 RootSiblings 也可以实现一个 Modal 组件，而且它的层级是在当前界面的最上层的。
## 实现界面 Render 相关
由于 RootSiblings 的实现是通过将组件添加到它注册到根节点中的，并不直接通过 Component 的 Render 进行布局，而 RN Modal 的显示隐藏是通过 visible 属性进行控制，所以在 `componentWillReceiveProps(nextProps)` 中根据 visible 进行相关的控制，部分实现代码如下
```
render() {
  if (this.props.visible) {
    this.RootSiblings && this.RootSiblings.update(this.renderRootSiblings());
  }
  return null;
}

componentWillReceiveProps(nextProps) {
    const { onShow, animationType, onDismiss } = this.props;
    const { visible } = nextProps;
    if (!this.RootSiblings && visible === true) { // 表示从没有到要显示了
      this.RootSiblings = new RootSiblings(this.renderRootSiblings(), () => {
        if (animationType === 'fade') {
          this._animationFadeIn(onShow);
        } else if (animationType === 'slide') {
          this._animationSlideIn(onShow);
        } else {
          this._animationNoneIn(onShow);
        }
      });
    } else if (this.RootSiblings && visible === false) { // 表示显示之后要隐藏了
      if (animationType === 'fade') {
        this._animationFadeOut(onDismiss);
      } else if (animationType === 'slide') {
        this._animationSlideOut(onDismiss);
      } else {
        this._animationNoneOut(onDismiss);
      }
   }
}
```
## 实现 Modal 展示动画相关
RN Modal 实现了三种动画模式，所以这里在通过 RootSiblings 实现 Modal 组件的时候也实现了这三种动画模式，这里借助的是 RN 提供的 Animated 和 Easing 进行相关的实现
- 'none' 这种不必多说，直接进行展示，没有动画效果
- 'fade' 淡入浅出动画，也就是透明度的一个变化，这里使用了 Easing.in 插值器使得效果更加平滑
- 'slide' 幻灯片的滑入画出动画，这是是组件 Y 方向位置的一个变化，这里使用了 Easing.in 插值器使得效果更加平滑

完整的一个 使用 RootSiblings 封装 Modal 实现代码如下
```
mport React, { Component } from 'react';
import {
  Animated, Easing, Dimensions, StyleSheet,
} from 'react-native';
import RootSiblings from 'react-native-root-siblings';

const { height } = Dimensions.get('window');
const animationShortTime = 250; // 动画时长为250ms

export default class ModalView extends Component {
  constructor(props) {
    super(props);
    this.state = {
      animationSlide: new Animated.Value(0),
      animationFade: new Animated.Value(0),
    };
  }

  render() {
    if (this.props.visible) {
      this.RootSiblings && this.RootSiblings.update(this.renderRootSiblings());
    }
    return null;
  }

  componentWillReceiveProps(nextProps) {
    const { onShow, animationType, onDismiss } = this.props;
    const { visible } = nextProps;
    if (!this.RootSiblings && visible === true) { // 表示从没有到要显示了
      this.RootSiblings = new RootSiblings(this.renderRootSiblings(), () => {
        if (animationType === 'fade') {
          this._animationFadeIn(onShow);
        } else if (animationType === 'slide') {
          this._animationSlideIn(onShow);
        } else {
          this._animationNoneIn(onShow);
        }
      });
    } else if (this.RootSiblings && visible === false) { // 表示显示之后要隐藏了
      if (animationType === 'fade') {
        this._animationFadeOut(onDismiss);
      } else if (animationType === 'slide') {
        this._animationSlideOut(onDismiss);
      } else {
        this._animationNoneOut(onDismiss);
      }
    }
  }

  renderRootSiblings = () => {
    return (
      <Animated.View style={[styles.root,
        { opacity: this.state.animationFade },
        {
          transform: [{
            translateY: this.state.animationSlide.interpolate({
              inputRange: [0, 1],
              outputRange: [height, 0],
            }),
          }],
        }]}>
        {this.props.children}
      </Animated.View>
    );
  }

  _animationNoneIn = (callback) => {
    this.state.animationSlide.setValue(1);
    this.state.animationFade.setValue(1);
    callback && callback();
  }

  _animationNoneOut = (callback) => {
    this._animationCallback(callback);
  }

  _animationSlideIn = (callback) => {
    this.state.animationSlide.setValue(0);
    this.state.animationFade.setValue(1);
    Animated.timing(this.state.animationSlide, {
      easing: Easing.in(),
      duration: animationShortTime,
      toValue: 1,
    }).start(() => callback && callback());
  }

  _animationSlideOut = (callback) => {
    this.state.animationSlide.setValue(1);
    this.state.animationFade.setValue(1);
    Animated.timing(this.state.animationSlide, {
      easing: Easing.in(),
      duration: animationShortTime,
      toValue: 0,
    }).start(() => this._animationCallback(callback));
  }

  _animationFadeIn = (callback) => {
    this.state.animationSlide.setValue(1);
    this.state.animationFade.setValue(0);
    Animated.timing(this.state.animationFade, {
      easing: Easing.in(),
      duration: animationShortTime,
      toValue: 1,
    }).start(() => callback && callback());
  }

  _animationFadeOut = (callback) => {
    this.state.animationSlide.setValue(1);
    this.state.animationFade.setValue(1);
    Animated.timing(this.state.animationFade, {
      easing: Easing.in(),
      duration: animationShortTime,
      toValue: 0,
    }).start(() => this._animationCallback(callback));
  }

  _animationCallback = (callback) => {
    this.RootSiblings && this.RootSiblings.destroy(() => {
      callback && callback();
      this.RootSiblings = undefined;
    });
  }
}
const styles = StyleSheet.create({
  root: {
    position: 'absolute',
    left: 0,
    top: 0,
    right: 0,
    bottom: 0,
  },
});
```
# 使用 View 封装 Modal
上面两种 Modal 的封装已经能够满足绝大部分业务场景的需求，但是如果在 Modal 中需要打开新的界面（不创建新的 ViewController 和 Acticity ），并且 Modal 不进行隐藏的话，比如使用 [react-navigation](https://reactnavigation.org/) 跳转页面，那么上面实现的 Modal 层级会太高了。所以这里通过 View 去实现类似的一个 Modal 控件。它的实现代码类似于上面的 RootSiblings 的实现。在 Render 中进行展示就好了，动画也可以使用上面的实现
```
 render() {
    return this._renderView()
 }
 
_renderView = () => {
    if (this.state.visible) {
      return (
        <Animated.View style={[styles.root,
          {opacity: this.state.animationFade},
          {
            transform: [{
              translateY: this.state.animationSlide.interpolate({
                inputRange: [0, 1],
                outputRange: [height, 0]
              }),
            }]
          }]}>
          {this.props.children}
        </Animated.View>
      );
    } else {
      return null
    }
}
```
# 整体 Modal 控件的封装
相对于 RN Modal ，上面新增了三种 Modal 的实现，为了整合整体的使用，所以需要对它们进行一个整体的封装使用，通过制定 modalType 的方式，来指定我们需要它内部的实现，先上代码
```
/**
 * @Author: linhe
 * @Date: 2019-05-12 10:11
 *
 * 因为ios端同时只能存在一个Modal，并且Modal多次显示隐藏会有很奇怪的bug
 *
 * 为了兼容ios的使用，这里需要封装一个ModalView
 *
 * Android 依旧使用 React Native Modal 来进行实现
 * ios 的话采用 RootSiblings 配合进行使用
 *
 * 这个是因为有的modal里面还需要跳转到其他界面
 * 这个时候主要要将该View放到最外边的层级才可以
 *
 * modalType:1 //表示使用Modal进行实现
 *           2 //表示使用RootSiblings进行实现
 *           3 //表示使用View进行实现
 * 注意:默认情况下 Android 使用的是1，ios使用的是2
 *
 * 同时采用与 React Native Modal 相同的API
 */
'use strict';
import React, {Component} from "react";
import {Animated, BackHandler, Platform, Easing, StyleSheet, Dimensions, Modal} from "react-native";
import PropTypes from 'prop-types'
import RootSiblings from 'react-native-root-siblings';
import FullScreenModal from './FullScreenModal/FullScreenModal'

const {height} = Dimensions.get('window')
const animationShortTime = 250 //动画时长为250ms
const DEVICE_BACK_EVENT = 'hardwareBackPress';

export default class ModalView extends Component {

  static propTypes = {
    isDarkMode: PropTypes.bool, // false 表示白底黑字，true 表示黑底白字
    autoKeyboard: PropTypes.bool, // 未知原因的坑，modal中的edittext自动弹起键盘要设置这个参数为true
    useReactModal: PropTypes.bool, // 是否使用 RN Modal 进行实现
    modalType: PropTypes.number // modalType 类型，默认 android 为 1，ios 为 2
  };

  static defaultProps = {
    isDarkMode: false,
    autoKeyboard: false,
    useReactModal: false,
    modalType: (Platform.OS === 'android' ? 1 : 2) // 默认 android 为1，ios 为2
  };

  constructor(props) {
    super(props);
    this.state = {
      visible: false,
      animationSlide: new Animated.Value(0),
      animationFade: new Animated.Value(0)
    };
  }

  render() {
    const {modalType} = this.props
    if (modalType === 1) { //modal实现
      return this._renderModal()
    } else if (modalType === 2) { //RootSiblings实现
      this.RootSiblings && this.RootSiblings.update(this._renderRootSiblings())
      return null
    } else { //View的实现
      return this._renderView()
    }
  }

  _renderModal = () => {
    const ModalView = this.props.useReactModal ? Modal : FullScreenModal
    return (
      <ModalView
        transparent={true}
        {...this.props}
        visible={this.state.visible}
        onRequestClose={() => {
          if (this.props.onRequestClose) {
            this.props.onRequestClose()
          } else {
            this.disMiss()
          }
        }}>
        {this.props.children}
      </ModalView>
    )
  }

  _renderRootSiblings = () => {
    return (
      <Animated.View style={[styles.root,
        {opacity: this.state.animationFade},
        {
          transform: [{
            translateY: this.state.animationSlide.interpolate({
              inputRange: [0, 1],
              outputRange: [height, 0]
            }),
          }]
        }]}>
        {this.props.children}
      </Animated.View>
    );
  }

  _renderView = () => {
    if (this.state.visible) {
      return (
        <Animated.View style={[styles.root,
          {opacity: this.state.animationFade},
          {
            transform: [{
              translateY: this.state.animationSlide.interpolate({
                inputRange: [0, 1],
                outputRange: [height, 0]
              }),
            }]
          }]}>
          {this.props.children}
        </Animated.View>
      );
    } else {
      return null
    }
  }

  show = (callback) => {
    if (this.isShow()) {
      return
    }
    const {modalType, animationType} = this.props
    if (modalType === 1) { //modal
      this.setState({visible: true}, () => callback && callback())
    } else if (modalType === 2) { //RootSiblings
      this.RootSiblings = new RootSiblings(this._renderRootSiblings(), () => {
        if (animationType === 'fade') {
          this._animationFadeIn(callback)
        } else if (animationType === 'slide') {
          this._animationSlideIn(callback)
        } else {
          this._animationNoneIn(callback)
        }
      });
      // 这里需要监听 back 键
      this._addHandleBack()
    } else { //view
      if (animationType === 'fade') {
        this.setState({visible: true}, () => this._animationFadeIn(callback))
      } else if (animationType === 'slide') {
        this.setState({visible: true}, () => this._animationSlideIn(callback))
      } else {
        this.setState({visible: true}, () => this._animationNoneIn(callback))
      }
      // 这里需要监听 back 键
      this._addHandleBack()
    }
  }

  disMiss = (callback) => {
    if (!this.isShow()) {
      return
    }
    const {modalType, animationType} = this.props
    if (modalType === 1) { //modal
      this.setState({visible: false}, () => callback && callback())
    } else { //RootSiblings和View
      if (animationType === 'fade') {
        this._animationFadeOut(callback)
      } else if (animationType === 'slide') {
        this._animationSlideOut(callback)
      } else {
        this._animationNoneOut(callback)
      }
      // 移除 back 键的监听
      this._removeHandleBack()
    }
  }

  isShow = () => {
    const {modalType} = this.props
    if (modalType === 1 || modalType === 3) { //modal和view
      return this.state.visible
    } else { //RootSiblings
      return !!this.RootSiblings
    }
  }

  _addHandleBack = () => {
    if (Platform.OS === 'ios') {
      return
    }
    // 监听back键
    this.handleBack = BackHandler.addEventListener(DEVICE_BACK_EVENT, () => {
      const {onRequestClose} = this.props
      if (onRequestClose) {
        onRequestClose()
      } else {
        this.disMiss()
      }
      return true
    });
  }

  _removeHandleBack = () => {
    if (Platform.OS === 'ios') {
      return
    }
    this.handleBack && this.handleBack.remove()
  }

  _animationNoneIn = (callback) => {
    this.state.animationSlide.setValue(1)
    this.state.animationFade.setValue(1)
    callback && callback()
  }

  _animationNoneOut = (callback) => {
    this._animationCallback(callback);
  }

  _animationSlideIn = (callback) => {
    this.state.animationSlide.setValue(0)
    this.state.animationFade.setValue(1)
    Animated.timing(this.state.animationSlide, {
      easing: Easing.in(),
      duration: animationShortTime,
      toValue: 1,
    }).start(() => callback && callback());
  }

  _animationSlideOut = (callback) => {
    this.state.animationSlide.setValue(1)
    this.state.animationFade.setValue(1)
    Animated.timing(this.state.animationSlide, {
      easing: Easing.in(),
      duration: animationShortTime,
      toValue: 0,
    }).start(() => this._animationCallback(callback));
  }

  _animationFadeIn = (callback) => {
    this.state.animationSlide.setValue(1)
    this.state.animationFade.setValue(0)
    Animated.timing(this.state.animationFade, {
      easing: Easing.in(),
      duration: animationShortTime,
      toValue: 1,
    }).start(() => callback && callback());
  }

  _animationFadeOut = (callback) => {
    this.state.animationSlide.setValue(1)
    this.state.animationFade.setValue(1)
    Animated.timing(this.state.animationFade, {
      easing: Easing.in(),
      duration: animationShortTime,
      toValue: 0,
    }).start(() => this._animationCallback(callback));
  }

  _animationCallback = (callback) => {
    if (this.props.modalType === 2) {//RootSiblings
      this.RootSiblings && this.RootSiblings.destroy(() => {
        callback && callback()
        this.RootSiblings = undefined
      })
    } else { //view
      this.setState({visible: false}, () => callback && callback())
    }
  }
}

const styles = StyleSheet.create({
  root: {
    position: 'absolute',
    left: 0,
    top: 0,
    right: 0,
    bottom: 0,
  }
});
```
这里主要通过 useReactModal 和 modalType 两个属性来控制我们所需要的实现
- modalType 表示 modal 内部实现方式，1 表示使用层级较高的实现，2 表示使用 RootSiblings 进行实现，3 表示使用 View 进行实现，当不进行指定的时候，默认 Android 为 1，ios 为 2
- useReactModal 主要针对 Android 端并且 modalType 为 1 的时候使用，true 表示使用 RN Modal，false 表示使用 FullScreenModal ，默认为 false
- 对外提供 show，disMiss，isShow 方法分别表示显示弹窗，隐藏弹窗和判断当前弹窗的状态，同时在 show 和 disMiss 方法调用的时候还添加了 callback 回调

# 其他
## Android Back 键的注意
当 Modal 组件使用 RootSiblings 或者 View 实现的时候，它并没有处理一个 Android 的返回键，所以对于这两种实现的时候，要额外处理一个 Back 键的操作，这里借助了 RN BackHandler 这个了，如果 modalType 不为 1 的话，需要通过 BackHandler 去实现一个返回键的监听，然后通过 onRequestClose 属性进行返回
```
const DEVICE_BACK_EVENT = 'hardwareBackPress';

_addHandleBack = () => {
    if (Platform.OS === 'ios') {
      return
    }
    // 监听back键
    this.handleBack = BackHandler.addEventListener(DEVICE_BACK_EVENT, () => {
      const {onRequestClose} = this.props
      if (onRequestClose) {
        onRequestClose()
      } else {
        this.disMiss()
      }
      return true
    });
}

_removeHandleBack = () => {
    if (Platform.OS === 'ios') {
      return
    }
    this.handleBack && this.handleBack.remove()
}
```
## View 封装 Modal 时候的注意
如果是 View 实现的 Modal 控件，那必须要注意它的一个层级，必须满足它所处于整个界面布局的最外层，否则它可能会被其他组件所挡住，同时它的最大的显示范围取决于它的父 View 的显示范围。
# 最后
当然在最后还是要附上实现效果图

![image](https://user-images.githubusercontent.com/4279515/59670834-2bd37380-91ef-11e9-84ba-237c02b09ea9.gif)

![image](https://user-images.githubusercontent.com/4279515/59670837-2d04a080-91ef-11e9-8bb3-52fa4eef7548.gif)

再一次附上 Demo 地址：[https://github.com/hzl123456/ModalViewDemo](https://github.com/hzl123456/ModalViewDemo)