# 一、uniApp 第一天

## 1. 新建项目

菜单：文件---新建---项目

<img src=".\media\image-20240618103727645.png" alt="image-20240618103727645" style="zoom: 50%;" />



## 2. 运行项目

菜单：运行---运行到浏览器---选择你安装的浏览器

注意：如果是第一次运行，需要加载一些基础插件，耐心等待；

<img src=".\media\image-20240618104224552.png" alt="image-20240618104224552" style="zoom:50%;" />

### 2.1 移动端模式

我们的项目默认是移动端项目，在访问的时候，切换到移动端设备：

1. 右键 ---  检查（开发者工具） 或者  F12；

2. 选中移动端模式：

   <img src=".\media\image-20240618104639046.png" alt="image-20240618104639046" style="zoom:50%;" />



## 3. 目录结构

我们真正的需要关注的只有pages目录和pages.json配置文件

<img src=".\media\image-20240618105542888.png" alt="image-20240618105542888" style="zoom:50%;" />

## 4. 创建页面

直接在pages目录创建页面即可：右键----新建页面

<img src=".\media\image-20240618112306097.png" alt="image-20240618112306097" style="zoom:50%;" />



## 5. pages.json 页面路由

对也页面进行一些配置：https://uniapp.dcloud.net.cn/collocation/pages.html

### 5.1 窗口配置

用于设置每个页面的状态栏、导航条、标题、窗口背景色等。

导航栏背景颜色：

```js
"navigationBarBackgroundColor":"16进制的颜色值"
```

导航栏标题颜色：

```js
"navigationBarTextStyle":"white or black"
```

导航栏标题：

```apl
"navigationBarTitleText":"标题文字"
```

是否显示导航栏的：

```js
"navigationStyle":"custom or default"
```

```
		{
			"path": "pages/index/index",
			"style": {
				"navigationBarTitleText": "uni-app",
				"navigationBarBackgroundColor":"#156165",
				"navigationBarTextStyle":"white",
				"navigationBarTitleText":"我的车牌识别项目",
				"navigationStyle":"default"
				
			}
		},
```



### 5.2 底部tab配置

**tab** 是指应用程序中的“标签页”或“选项卡”。多 tab 应用就是一个有多个标签页的应用，每个标签页对应一个独立的页面，当用户点击某个标签页时，显示对应的页面内容。

**tabBar** 是“标签栏”或“选项卡栏”。在 pages.json 中提供 tabBar 配置，不仅仅是为了方便快速开发导航，更重要的是在App和小程序端提升性能。在这两个平台，底层原生引擎在启动时无需等待js引擎初始化，即可直接读取 pages.json 中配置的 tabBar 信息，渲染原生tab。

图标下载地址：https://www.iconfont.cn/

配置代码参考如下：只需要关注tabBar配置

```json
{
	"pages": [ //pages数组中第一项表示应用启动页，参考：https://uniapp.dcloud.io/collocation/pages
		{
			"path": "pages/index/index",
			"style": {
				"navigationBarTitleText": "uni-app",
				"navigationBarBackgroundColor":"#156165",
				"navigationBarTextStyle":"white",
				"navigationBarTitleText":"我的车牌识别项目",
				"navigationStyle":"default"
				
			}
		},
		{
			"path" : "pages/my_try/my_try",
			"style" : 
			{
				"navigationBarTitleText" : "个人中心",
				"navigationBarBackgroundColor":"#000000",
				"navigationBarTextStyle":"white"
			}
		},
		{
			"path" : "pages/ai/ai",
			"style" : 
			{
				"navigationBarTitleText" : "车牌识别",
				"navigationBarBackgroundColor":"#000000",
				"navigationBarTextStyle":"white"
			}
		}

	],
	"globalStyle": {
		"navigationBarTextStyle": "black",
		"navigationBarTitleText": "uni-app",
		"navigationBarBackgroundColor": "#F8F8F8",
		"backgroundColor": "#F8F8F8"
	},
	"uniIdRouter": {},
	"tabBar": {
		"color": "#7A7E83",
		"selectedColor": "#3cc51f",
		"borderStyle": "black",
		"backgroundColor": "#ffffff",
		"list": [{
			"pagePath": "pages/index/index",
			"iconPath": "static/home.png",
			"selectedIconPath": "static/home2.png",
			"text": "首页"
		}, {
			"pagePath": "pages/my_try/my_try",
			"iconPath": "static/my.png",
			"selectedIconPath": "static/my2.png",
			"text": "我的"
		},
		{
			"pagePath": "pages/ai/ai",
			"iconPath": "static/car.png",
			"selectedIconPath": "static/car2.png",
			"text": "car"
		}]
	}

}
```



## 6. 项目发布

在HBuilderX工具栏，点击发行，选择原生app-云端打包，如下图：

![img](.\media\uni20190222-11.png)



### 6.1 打包设置

我们只打包Android：

<img src=".\media\image-20240618134500397.png" alt="image-20240618134500397" style="zoom: 50%;" />

### 6.2 准备工作

1. 需要注册为dcloud.net.cn的会员并登录；

2. 需要认证手机号：如果没有认证，会有提示；

3. 第一次打包，需要加载很多底座（模块），耐心等待；

   

## 7. 页面实现

### 7.1 结构层

就是写组件：就是用<>括起来的

#### 7.1.1 view

用uv快速定义

class定义属性

试图组件view，用于包裹各种元素内容：

**属性说明**

| 属性名                 | 类型    | 默认值 | 说明                                                         |
| :--------------------- | :------ | :----- | :----------------------------------------------------------- |
| hover-class            | String  | none   | 指定按下去的样式类。当 hover-class="none" 时，没有点击态效果 |
| hover-stop-propagation | Boolean | false  | 指定是否阻止本节点的祖先节点出现点击态，App、H5、支付宝小程序、百度小程序不支持（支付宝小程序、百度小程序文档中都有此属性，实测未支持） |
| hover-start-time       | Number  | 50     | 按住后多久出现点击态，单位毫秒                               |
| hover-stay-time        | Number  | 400    | 手指松开后点击态保留时间，单位毫秒                           |

```html
	<view class="avatar"> </view>
	
	<view class="username"> </view>
	
	<view class="tel"> </view>
```

#### 7.1.2 image

图片组件。

```JS
	<view class="pic">
		<image src="https://q5.itc.cn/q_70/images03/20240530/eabf049838e644abafca8d00bc1bb0a2.png" mode=""></image>
	</view>
```

| 属性名                 | 类型        | 默认值        | 说明                                                         | 平台差异说明                                   |
| :--------------------- | :---------- | :------------ | :----------------------------------------------------------- | :--------------------------------------------- |
| src                    | String      |               | 图片资源地址                                                 |                                                |
| mode                   | String      | 'scaleToFill' | 图片裁剪、缩放的模式                                         |                                                |
| lazy-load              | Boolean     | false         | 图片懒加载。只针对page与scroll-view下的image有效             | 微信小程序、百度小程序、抖音小程序、飞书小程序 |
| fade-show              | Boolean     | true          | 图片显示动画效果                                             | 仅App-nvue 2.3.4+ Android有效                  |
| webp                   | boolean     | false         | 在系统不支持webp的情况下是否单独启用webp。默认false，只支持网络资源。webp支持详见下面说明 | 微信小程序2.9.0                                |
| show-menu-by-longpress | boolean     | false         | 开启长按图片显示识别小程序码菜单                             | 微信小程序2.7.0                                |
| draggable              | boolean     | true          | 是否能拖动图片                                               | H5 3.1.1+、App（iOS15+）                       |
| @error                 | HandleEvent |               | 当错误发生时，发布到 AppService 的事件名，事件对象event.detail = {errMsg: 'something wrong'} |                                                |
| @load                  | HandleEvent |               | 当图片载入完毕时，发布到 AppService 的事件名，事件对象event.detail = {height:'图片高度px', width:'图片宽度px'} |                                                |

#### 7.1.3 swiper

滑块视图容器。

一般用于左右滑动或上下滑动，比如banner轮播图。

注意滑动切换和滚动的区别，滑动切换是一屏一屏的切换。swiper下的每个swiper-item是一个滑动切换区域，不能停留在2个滑动区域之间。

```JS
	<swiper class="swiper" :indicator-dots="false" :autoplay="true" :interval="3000" :duration="1000">
		<swiper-item>
			<image
				src=""
		</swiper-item>
		<swiper-item>
			<view class="swiper-item">
				<image src="https://img1.baidu.com/it/u=396994792,3517834831&fm=253&fmt=auto&app=138&f=JPEG?w=889&h=500"
					mode=""></image>
			</view>
		</swiper-item>
		<swiper-item>
			<view class="swiper-item">
				<image
					src="https://img1.baidu.com/it/u=1424252583,1885375173&fm=253&fmt=auto&app=138&f=JPEG?w=998&h=500"
					mode=""></image>
			</view>
		</swiper-item>
	</swiper>

```

**`:indicator-dots="false"`**：

- 控制轮播图底部的指示点是否显示。
- `false` 表示不显示指示点。

**`:autoplay="true"`**：

- 控制轮播图是否自动播放。
- `true` 表示轮播图会自动切换幻灯片。

**`:interval="3000"`**：

- 设置自动切换的时间间隔，单位为毫秒。
- `3000` 表示每隔 3 秒切换一张幻灯片。

**`:duration="1000"`**：

- 设置滑动动画的持续时间，单位为毫秒。
- `1000` 表示滑动动画持续 1 秒。

**属性说明**

| 属性名                         | 类型        | 默认值            | 说明                                                         | 平台差异说明                                                 |
| :----------------------------- | :---------- | :---------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| indicator-dots                 | Boolean     | false             | 是否显示面板指示点                                           |                                                              |
| indicator-color                | Color       | rgba(0, 0, 0, .3) | 指示点颜色                                                   |                                                              |
| indicator-active-color         | Color       | #000000           | 当前选中的指示点颜色                                         |                                                              |
| active-class                   | String      |                   | swiper-item 可见时的 class                                   | 支付宝小程序                                                 |
| changing-class                 | String      |                   | acceleration 设置为 true 时且处于滑动过程中，中间若干屏处于可见时的class | 支付宝小程序                                                 |
| autoplay                       | Boolean     | false             | 是否自动切换                                                 |                                                              |
| current                        | Number      | 0                 | 当前所在滑块的 index                                         |                                                              |
| current-item-id                | String      |                   | 当前所在滑块的 item-id ，不能与 current 被同时指定           | 支付宝小程序不支持                                           |
| interval                       | Number      | 5000              | 自动切换时间间隔                                             |                                                              |
| duration                       | Number      | 500               | 滑动动画时长                                                 | app-nvue不支持                                               |
| circular                       | Boolean     | false             | 是否采用衔接滑动，即播放到末尾后重新回到开头                 |                                                              |
| vertical                       | Boolean     | false             | 滑动方向是否为纵向                                           |                                                              |
| previous-margin                | String      | 0px               | 前边距，可用于露出前一项的一小部分，接受 px 和 rpx 值        | app-nvue、抖音小程序、飞书小程序不支持                       |
| next-margin                    | String      | 0px               | 后边距，可用于露出后一项的一小部分，接受 px 和 rpx 值        | app-nvue、抖音小程序、飞书小程序不支持                       |
| acceleration                   | Boolean     | false             | 当开启时，会根据滑动速度，连续滑动多屏                       | 支付宝小程序                                                 |
| disable-programmatic-animation | Boolean     | false             | 是否禁用代码变动触发 swiper 切换时使用动画。                 | 支付宝小程序                                                 |
| display-multiple-items         | Number      | 1                 | 同时显示的滑块数量                                           | app-nvue、支付宝小程序不支持                                 |
| skip-hidden-item-layout        | Boolean     | false             | 是否跳过未显示的滑块布局，设为 true 可优化复杂情况下的滑动性能，但会丢失隐藏状态滑块的布局信息 | App、微信小程序、京东小程序                                  |
| disable-touch                  | Boolean     | false             | 是否禁止用户 touch 操作                                      | App 2.5.5+、H5 2.5.5+、支付宝小程序、抖音小程序与飞书小程序（只在初始化时有效，不能动态变更） |
| touchable                      | Boolean     | true              | 是否监听用户的触摸事件，只在初始化时有效，不能动态变更       | 抖音小程序与飞书小程序（uni-app 2.5.5+ 推荐统一使用 disable-touch） |
| easing-function                | String      | default           | 指定 swiper 切换缓动动画类型，有效值：default、linear、easeInCubic、easeOutCubic、easeInOutCubic | 微信小程序、快手小程序、京东小程序                           |
| @change                        | EventHandle |                   | current 改变时会触发 change 事件，event.detail = {current: current, source: source} |                                                              |
| @transition                    | EventHandle |                   | swiper-item 的位置发生改变时会触发 transition 事件，event.detail = {dx: dx, dy: dy}，支付宝小程序暂不支持dx, dy | App、H5、微信小程序、支付宝小程序、抖音小程序、飞书小程序、QQ小程序、快手小程序 |
| @animationfinish               | EventHandle |                   | 动画结束时会触发 animationfinish 事件，event.detail = {current: current, source: source} | 抖音小程序与飞书小程序不支持                                 |

**组件的关系：**

1. 父子关系
2. 兄弟关系
3. 后代关系

​	

### 7.2 表现层

#### 7.2.1 选择器

选择器的作用：把定义的样式和对应的组件联系起来。



**class：**表示一类，只需要记住一点（.），和组件的class属性值进行对应         **类选择器**

```css
	.cartext {
			color: black; /* 设置颜色为深蓝色 */
			font-weight: bold; /* 加粗 */
			text-align: center; /* 居中对齐 */
			font-size: 16px; /* 放大字体 */
			margin-top: 10px; /* 增加上方间距 */
			margin-bottom: 10px; /* 增加下方间距 */
	}
```



tag和后代选择器

```css
/* Swiper容器样式 */
	swiper {
		width: 100%;
		background-color: aquamarine;
}
	.swiper-item image{
		width: 100%;
		height: 150px;
	}
	
-------------

	.top .student{
		color: blueviolet;
		font-size: 16px;
		text-align: center;
	}


	.middle image {
		width: 100%;
		height: 500px; 
}

```





#### 7.2.2 选择器权重

在冲突的情况之下：

1. class  > tag；

2. 就近原则：在后面的覆盖前面的样式

3. 组合选择器权重会相加；

   ```
   class:10
   tag:01
   class tag : 10 + 01 = 11
   
   .stu view{}
   ```

   



####  7.2.3 布局的本质是盒子

页面布局的本质是盒子:

1. 组件就是盒子；
2. 盒子就是组件；

```tex
电视  ----> 电视墙 /电视柜
衣服  ----> 衣柜 / 衣帽间
电脑  ----> 电脑桌
```

**实例：**

```javascript
<template>
	<view class="menu">
		<view class="left">
			<view class="logo1">
				<image src="../../static/logo1.png" mode=""></image>
				
			</view>
			<view class="logo2">
				<image src="data:image……AABJRU5ErkJggg==" mode=""></image>
				
			</view>
		</view>

		<view class="right">
			登陆/注册
		</view>
		
	</view>
	<view class="video">
		<view class="video">
			<view class="title">
				校园风光
			</view>
			<view class="list">
				<view class="v">
					<view class="vi">
						<image src="../../static/1618562756_5619_thumb.jpg" mode=""></image>
					</view>
					<view class="vi_text">
						北京工业大学
					</view>
				</view>
				<view class="v">
					<view class="vi">
						<image src="../../static/1618562756_5619_thumb.jpg" mode=""></image>
					</view>
					<view class="vi_text">
						北京工业大学
					</view>
				</view>
				<view class="v">
					<view class="vi">
						<image src="../../static/1618562756_5619_thumb.jpg" mode=""></image>
					</view>
					<view class="vi_text">
						北京工业大学
					</view>
				</view>
				<view class="v">
					<view class="vi">
						<image src="../../static/1618562756_5619_thumb.jpg" mode=""></image>
					</view>
					<view class="vi_text">
						北京工业大学
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				
			}
		},
		methods: {
			
		}
	}
</script>

<style>
	.menu{
		/* 盒子自身的问题 */
		/* width: 100%; pc端是375  */
		width: 750rpx;
		height: 200rpx;
/* 		height: 100px; */
		background: linear-gradient(180deg, rgba(255, 243, 234, .86), #fffcfb 55.73%, hsla(0, 0%, 100%, 0));
		/* 弹性盒子 */
		display: flex;
		/* 控制在水平方向上的对齐方式 */
		justify-content: space-between;
		/* 控制在垂直方向上的对齐方式 */
		align-items: center;
		/* 靠边太近了:内边距 第一个参数是上下 第二个是左右 */
		padding: 0 20rpx;
		/* 把标准盒子转为怪异盒子 为了简化布局和避免意外的尺寸变化 */
		box-sizing: border-box;
	}
	.menu .left{
		/* 弹性盒子 */
		display: flex;
		/* 控制在水平方向上的对齐方式 */
		justify-content: space-between;
		/* 控制在垂直方向上的对齐方式 */
		align-items: center;
	}
	.logo1 image{
		width: 276rpx;
		height: 58rpx;
	}
	.logo2 image{
		width: 188rpx;
		height: 54rpx;
	}
	.menu .right{
		    width: 174rpx;
		    height: 56rpx;
			/* 一行文本的高度：让文字在垂直方向居中的效果 */
		    line-height: 56rpx;
		    background: linear-gradient(280deg, #ff7e27, #fd9f49 98%);
			/* 定义圆角 */
		    border-radius: 8rpx;
		    text-align: center;
		    color: #fff;
	}

	.video{
		/* 靠边太近了:内边距 第一个参数是上下 第二个是左右 */
		padding: 20 20rpx;
		/* 把标准盒子转为怪异盒子 */
		box-sizing: border-box;
	}
	.video .vi{
		width:345rpx ;
		height:191.7rpx;
		background-color: #fd9f49;
		margin-bottom: 10rpx;
	}
	.video .list{
		display: flex;
		justify-content: space-between;
		align-items: center;
		/* 放不下的情况下是否换行 */
		flex-wrap: wrap;
	}
	.video .v image{
		width: 100%;
		height:191.7rpx;
	}
	.video .v .vi_text{
		height: 44rpx;
		line-height: 44rpx;
		text-align: center;
		/* 超出部分显示省略号 */
		text-overflow: ellipsis;

	}
	.video .title{
		font-weight: 600;
		font-size: 20px;
		color: #333;
		padding: 20rpx;
	}
</style>
```

### 7.3 行为层

# 二、uniApp 第二天

## 1. 行为层

https://uniapp.dcloud.net.cn/tutorial/vue3-basics.html#%E6%A8%A1%E6%9D%BF%E8%AF%AD%E6%B3%95

### 1.1 响应性特征

数据具备响应性特征：当数据发生改变的时候，页面会自动重新渲染。

总结：任何效果的实现，都是通过修改数据来实现。



### 1.2 数据的定义

一定要把数据放到data方法里面，只有放在data方法里面才具备响应性特征：

```js
export default {
    // 数据方法
    data() {
        return {
            // 数据一定要放在这里
            username:'HeTao',
            age:20,
            tel:'15982036598'
        }
    },

    methods: {

    }
}
```



### 1.3 数据的绑定

#### 1.3.1 文本插值

```vue
<template>
	<view>
		<view class="name">
			姓名：{{username}}
		</view>
		<view class="age">
			年龄：{{age}}
		</view>
		<view class="tel">
			手机号：{{tel}}
		</view>
		<view class="gender">
			性别：{{genderArr[1]}}
		</view>
	</view>
</template>

<script>
	export default {
		// 数据方法
		data() {
			return {
				// 数据一定要放在这里
				username: 'HeTao',
				age: 20,
				tel: '15982036598',
				genderArr:['保密', '帅哥', '美女']
			}
		},

		methods: {

		}
	}
</script>

```



####  1.3.2 属性值绑定

主要用于组件的属性值的控制，如image的src，所有的组件的所有的属性都可以进行属性值绑定

**使用:进行属性值绑定**：这是简写的方式，全称时 v-bind:

```vue
<template>
	<view>
		<view class="avatar">
			<image :src="avatar" mode=""></image>
		</view>
	</view>
</template>

<script>
	export default {
		// 数据方法
		data() {
			return {
				avatar:'https://oimagec7.ydstatic.com/image?id=-7601312740411526213&product=xue'
			}
		},
        
		methods: {
		}
	}
</script>
```



#### 1.3.3 事件绑定

使用v-on:来绑定事件，但是我们经常会**简写为 @** 

```vue
<template>
	<view>
		<view class="name">
			姓名：{{username}}
		</view>
		<view class="age">
			年龄：{{age}}
		</view>
		<view class="tel">
			手机号：{{tel}}
		</view>
		<view class="gender">
			性别：{{genderArr[gender]}}
		</view>
		<view class="avatar">
			<image :src="avatar" mode=""></image>
		</view>
	</view>

//tap是触摸的意思
	<button type="default" @tap="doWhat">修改头像</button>
</template>

<script>
	export default {
		// 数据方法
		data() {
			return {
				// 数据一定要放在这里：会追加到this身上
				username: 'HeTao',
				age: 20,
				tel: '15982036598',
				gender: 1,
				genderArr: ['保密', '帅哥', '美女'],
				avatar: 'https://oimagec7.ydstatic.com/image?id=-7601312740411526213&product=xue'
			}
		},

		methods: {
			doWhat() {
				console.log('触摸事件发生')
				// 想做点什么业务
				console.log(this)
				this.avatar = 'https://oimagea2.ydstatic.com/image?id=8282417838083585413&product=xue'
				this.username = 'LiuJiaWei'
				this.age = 24
				this.gender = 0 
			}
		}
	}
</script>

```



#### 1.3.4 列表渲染

```vue
<template>
	<view class="list">
		<view class="item" v-for="(p, i) in cars">
			{{i}}-{{p.name}}， 价格：{{p.price}}， 购买数量：{{p.quantity}}
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				cars: [{
						id: 1,
						name: '商品1',
						price: 49.99,
						quantity: 3
					},
					{
						id: 2,
						name: '商品2',
						price: 24.99,
						quantity: 1
					},
					{
						id: 3,
						name: '商品3',
						price: 9.99,
						quantity: 2
					},
					{
						id: 4,
						name: '商品4',
						price: 14.99,
						quantity: 5
					},
					{
						id: 5,
						name: '商品5',
						price: 39.99,
						quantity: 1
					}					
				]
			}
		},
		methods: {

		}
	}
</script>

<style>

</style>
```



#### 1.3.5 条件渲染

```vue
<template>
	<view class="unlogin" v-if="!islogin">
		登录之前的登录表单
		<button type="warn" @tap="userLogin">登录</button>
	</view>
	
	
	<view class="logined" v-else>
		登录之后的信息
	</view>
	
</template>

<script>
	export default {
		// 数据方法
		data() {
			return {
				islogin: false
            }
		},

		methods: {
			userLogin(){
				this.islogin = true
			}
		}
	}
</script>

<style>

</style>

```

#### 1.3.6 双向数据绑定

使用v-model实现双向数据绑定

```vue
<template>
	<view class="unlogin" v-if="!islogin">
		<uni-section title="用户登录" type="line">
			<view class="example">
				<uni-forms ref="valiForm" :modelValue="logindata">
					<uni-forms-item label="手机号" required name="tel">
						<uni-easyinput type="number" v-model="logindata.tel" placeholder="请输入手机号" />
					</uni-forms-item>
					<uni-forms-item label="密码" required name="passwd">
						<uni-easyinput type="password" v-model="logindata.passwd" placeholder="请输入密码" />
					</uni-forms-item>
				</uni-forms>
				<button type="primary" @tap="userLogin">提交</button>
			</view>
		</uni-section>
	</view>

	<view class="logined" v-else>

		<view class="name">
			姓名：{{username}}
		</view>
		<view class="age">
			年龄：{{age}}
		</view>
		<view class="tel">
			手机号：{{tel}}
		</view>
		<view class="gender">
			性别：{{genderArr[gender]}}
		</view>
		<view class="avatar">
			<image :src="avatar" mode=""></image>
		</view>
		<button type="default" @tap="doWhat">修改头像</button>
	</view>

</template>

<script>
	export default {
		// 数据方法
		data() {
			return {
				// 校验表单数据
				logindata: {
					tel: '',
					passwd: '',
				},

				islogin: false,
				// 数据一定要放在这里：会追加到this身上
				username: 'HeTao',
				age: 20,
				tel: '15982036598',
				gender: 1,
				genderArr: ['保密', '帅哥', '美女'],
				avatar: 'https://oimagec7.ydstatic.com/image?id=-7601312740411526213&product=xue'
			}
		},

		methods: {
			doWhat() {
				console.log('触摸事件发生')
				// 想做点什么业务
				console.log(this)
				this.avatar = 'https://oimagea2.ydstatic.com/image?id=8282417838083585413&product=xue'
				this.username = 'LiuJiaWei'
				this.age = 24
				this.gender = 0

			},
			userLogin() {
				// this.islogin = true
				let data = this.logindata
				console.log(data)
			}
		}
	}
</script>

<style>
	.example {
		padding: 15px;
		background-color: #fff;
	}

	.segmented-control {
		margin-bottom: 15px;
	}

	.button-group {
		margin-top: 15px;
		display: flex;
		justify-content: space-around;
	}

	.form-item {
		display: flex;
		align-items: center;
	}

	.button {
		display: flex;
		align-items: center;
		height: 35px;
		margin-left: 10px;
	}
</style>
```



### 2.4 API 接口

####  2.4.1 网络请求

```js
uni.request({
    url: 'https://www.example.com/request', //仅为示例，并非真实接口地址。
    data: {
        text: 'uni.request'
    },
    header: {
        'custom-header': 'hello' //自定义请求头信息
    },
    success: (res) => {
        console.log(res.data);
        this.text = 'request success';
    }
});

```

#### 2.4.2 文件上传

```js
uni.chooseImage({
	success: (chooseImageRes) => {
		const tempFilePaths = chooseImageRes.tempFilePaths;
		uni.uploadFile({
			url: 'https://www.example.com/upload', //仅为示例，非真实的接口地址
			filePath: tempFilePaths[0],
			name: 'file',
			formData: {
				'user': 'test'
			},
			success: (uploadFileRes) => {
				console.log(uploadFileRes.data);
			}
		});
	}
});

```

#### 2.4.3 页面跳转

```js
//在起始页面跳转到test.vue页面并传递参数
uni.navigateTo({
	url: 'test?id=1&name=uniapp'
});

// 跳转到 tabBar 页面，并关闭其他所有非 tabBar 页面。
uni.switchTab({
	url: '/pages/index/index'
});
```



#### 2.4.4 页面交互

https://uniapp.dcloud.net.cn/api/ui/prompt.html

```js
uni.showToast({
	title: '标题',
	duration: 2000
});

```



#### 2.4.5 数据缓存

https://uniapp.dcloud.net.cn/api/storage/storage.html

```js
uni.setStorage({
	key: 'storage_key',
	data: 'hello',
	success: function () {
		console.log('success');
	}
});

```



