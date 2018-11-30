# 1.Angular2的按需加载及延迟加载
## 1.1 根模块AppModule
- 假设按需加载的模块为LazyModule，首页模块为LoginModule，根模块为AppModule。首先在AppModule.ts中去掉LazyModule的引入。
```
import { NgModule } from '@angular/core';
import { BrowserModule }  from '@angular/platform-browser';
import { HttpModule} from '@angular/http';
import {RouterModule} from '@angular/router';

//import { LazyModule} from './lazy/lazy.module';
import { LoginModule} from './login/login.module';
import {AppRoutes} from './app.routes';

@NgModule({
  imports: [
    BrowserModule,
    HttpModule,
    RouterModule.forRoot(AppRoutes)
    //LazyModule,  //去掉按需加载模块    
    LoginModule,
  ],
  declarations: [
    AppComponent
  ],
  providers:[HttpService],
  bootstrap: [ AppComponent ]
})
export class AppModule { }
```
## 1.2 惰性路由
- 在根路由文件AppRoutes.ts文件中加入Lazy模块
```
import {Route} from '@angular/router';
import {LoginRoutes} from './login/login.routes';
//去掉Lazy模块的路由引用
//import {LazyRoutes} from './home/home.routes';


export const AppRoutes: Route[] = [
    {
        path:'login',
        component:LoginRoutes
    },
    {
        path:'homepage',
        //注意：此处算是按需加载路由的关键点之一
        loadChildren: 'app/lazy/lazy.module#LazyModule'
    },
    {
        path:'**',
        redirectTo: 'login'
    }
];
```
- Lazy模块路由文件（LazyRoutes.ts）
```
import {Route} from '@angular/router';
import {LazyComponent} from './home.component'; 


export const LazyRoutes: Route[] = [
    {
        //注意：此处不能写成path:'lazy'
        path:'',
        component: LazyComponent
    }
];
```
- Lazy模块路由引入
```
import { NgModule } from '@angular/core';
import { RouterModule } from '@angular/router';
import { LazyComponent } from './home.component';
import { LazyRoutes } from './home.routes';

@NgModule({
  imports: [
    CommonModule,
    //路由模块引入
    RouterModule.forChild(LazyRoutes)
  ],
  declarations: [LazyComponent],
  exports: [LazyComponent]
})
export class LazyModule { }
```
## 1.3 预加载
- 项目在按需加载后，有些模块在后期必然会被访问到，于是可以设置延时加载。系统在优先显示首页后，在后台默默加载设置了预加载的模块。在这里简单设置为所有模块都预加载。
```
RouterModule.forRoot(
      AppRoutes,
      {preloadingStrategy:PreloadAllModules}),//预加载
//注意：路由从之前的编写方式改为按需加载模式时，切记要在子路由里删掉原路由（当初探索时在这个坑里爬了好久）。      
```
# 2.webpack使用
```
            /**
             * awesome-typescript-loader 用来配合tsconfig编译typescript
             * angular2-template-loader 用来把模块的样式和html打到component中， inlines all html and style's in angular2 components
             * angular2-router-loader lazy-load 跟之前的懒加载类似，从而写法上简单很多
             */
            {
                test: /\.ts$/,
                loaders: [
                    'awesome-typescript-loader',
                    'angular2-template-loader',
                    'angular2-router-loader'],//关键的loader
                exclude: [/\.(spec|e2e)\.ts$/]
            }
```
# 3.公共模块jquery引用方式
- 许多npm包都是基于jquery的，所以jquery的引入极为重要，需要全局引入。在webpack配置文件中插件 里引入，方法如下：
```
plugins: [
      //全局挂载jquery，并且要在tsconfig.json中配置jquery
     new webpack.ProvidePlugin({
      $:"jquery",
      jQuery:"jquery",
      "window.jQuery":"jquery"
    }),
    ...]
```
- 同时需要在tsconfig.json配置文件中注明jquery—–此处很关键
```
"type":[
      "jquery"
    ]
```
