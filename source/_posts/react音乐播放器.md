---
title: react音乐播放器
date: 2018-04-25 20:55:34
categories: 技术
tags: react
---

在用**hexo**搭建博客的时候，想在页面上展示一首音乐。一开始想到的是直接用网易云的外链播放，本质是在页面上加一个iframe。然而很局限的是，由于现在的版权风暴，许多歌曲已经下架了，而自己加入网易云盘的歌曲，虽然可以自己听，但是并不能用作外链（cookie检查）。

于是想动手做一个h5的页面音乐播放器。在考虑了原生H5之后，发现不太方便，因为引入hexo的话，需要组件化，而原生H5对前端技术要求比较高。于是采用了**ReactJS**。

ReactJS一个优点在于，组件化的思想，让不擅长前端的童鞋，也能用组装的方式去完成一个控件。

 1. 使用npm搭建React开发环境
 只引用react即可，并未采用前端工程化打包webpack。因为我们只需要打包好控件，然后引用js，css即可。

 2. UI。主要是先在入口的index.js里面把控件的结构构建好，然后设计对应的css样式。这样我们的控件就有了一个大体的模样了。
![样式示例][1]
 
 3. 逻辑处理。在React的生命周期里，使用 getDefaultProps 方法设置初始的参数，即播放歌曲的信息（之后会考虑读取配置文件）。然后在render方法中对属性值进行事件的操作。比如播放，暂停，下一首...

```javascript
	render: function() {		
		return (
			<div className="musicplayer">
							

				{/* 音乐专辑图  */}
				<TrackInfo track={this.props.tracks[this.state.currentTrackIndex]} />

				{/* 音乐信息   */}
				<Progress progress={this.state.currentTime / this.state.currentTotalTime * 100 + '%'} />

				{/* 播放进度条  */}
				<Controls isPlay={this.state.playStatus} onPlay={this.play} onPrevious={this.previous} onNext={this.next} />

				{/* 播放时间   */}
				<Time currentTime={this.state.currentTime} currentTotalTime={this.state.currentTotalTime} />

				{/* 音频控件  */}
				<audio id="audio" src={this.props.tracks[this.state.currentTrackIndex].mp3Url}></audio>
			</div>
		);
	}
```

 打包使用，使用npm run build 构建完成，提取出js，和css，在需要使用的页面上 插入div，命名为reactmusicplayer即可。
 
 ![样例][2]
 
由于缺乏前端功底，实在是不好看，所以还是向网易云妥协了。

**PS： 本地调试的时候，localhost请求个人网站上的静态json文件来完成配置时，遇到了jsonp的ajax跨域问题。将在下阶段研究一哈！**
 

 

 


  [1]: https://blog.felixplanet.cn/images/sample/reactpic1.png
  [2]: https://blog.felixplanet.cn/images/sample/reactpic2.png