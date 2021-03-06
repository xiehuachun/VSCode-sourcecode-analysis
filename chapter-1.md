# VSCode源码分析 - 主启动流程
## 目录
* <a href="https://github.com/fzxa/VSCode-sourcecode-analysis/blob/master/chapter-1.md">启动主进程</a>
* <a href="https://github.com/fzxa/VSCode-sourcecode-analysis/blob/master/chapter-2.md">实例化服务</a>
* <a href="https://github.com/fzxa/VSCode-sourcecode-analysis/blob/master/chapter-3.md">事件分发</a>
* <a href="https://github.com/fzxa/VSCode-sourcecode-analysis/blob/master/chapter-4.md">进程通信</a>
* <a href="https://github.com/fzxa/VSCode-sourcecode-analysis/blob/master/chapter-5.md">主要窗口</a>
* <a href="https://github.com/fzxa/VSCode-sourcecode-analysis/blob/master/chapter-6.md">开发调试</a>

## <a name="1">简介</a>
Visual Studio Code(简称VSCode) 是开源免费的IDE编辑器，原本是微软内部使用的云编辑器(Monaco)。

git仓库地址： https://github.com/microsoft/vscode

通过Eletron集成了桌面应用，可以跨平台使用，开发语言主要采用微软自家的TypeScript。
整个项目结构比较清晰，方便阅读代码理解。成为了最流行跨平台的桌面IDE应用

微软希望VSCode在保持核心轻量级的基础上，增加项目支持，智能感知，编译调试。
![img](https://github.com/fzxa/VSCode-sourcecode-analysis/blob/master/vscode-vside-zh.png)

## 编译安装
下载最新版本,目前我用的是1.37.1版本
官方的wiki中有编译安装的说明  [How to Contribute](https://github.com/microsoft/vscode/wiki/How-to-Contribute?_blank)

Linux, Window, MacOS三个系统编译时有些差别，参考官方文档，
在编译安装依赖时如果遇到connect timeout, 需要进行科学上网。

> 需要注意的一点 运行环境依赖版本 Nodejs x64 version >= 10.16.0, < 11.0.0,  python 2.7(3.0不能正常执行)


## <a name="2">技术架构</a>
![img](https://github.com/fzxa/VSCode-sourcecode-analysis/blob/master/vscode-framework.png)

### Electron 

> [Electron](https://electronjs.org/?_blank) 是一个使用 JavaScript, HTML 和 CSS 等 Web 技术创建原生程序的框架，它负责比较难搞的部分，你只需把精力放在你的应用的核心上即可 (Electron = Node.js + Chromium + Native API)

### Monaco Editor
> [Monaco Editor](https://github.com/microsoft/monaco-editor?_blank)是微软开源项目, 为VS Code提供支持的代码编辑器，运行在浏览器环境中。编辑器提供代码提示，智能建议等功能。供开发人员远程更方便的编写代码，可独立运行。

### TypeScript
> TypeScript是一种由微软开发的自由和开源的编程语言。它是JavaScript的一个超集，而且本质上向这个语言添加了可选的静态类型和基于类的面向对象编程

## 目录结构
```
├── build         # gulp编译构建脚本
├── extensions    # 内置插件
├── product.json  # App meta信息
├── resources     # 平台相关静态资源
├── scripts       # 工具脚本，开发/测试
├── src           # 源码目录
└── typings       # 函数语法补全定义
└── vs
    ├── base        # 通用工具/协议和UI库
    │   ├── browser # 基础UI组件，DOM操作
    │   ├── common  # diff描述，markdown解析器，worker协议，各种工具函数
    │   ├── node    # Node工具函数
    │   ├── parts   # IPC协议（Electron、Node），quickopen、tree组件
    │   ├── test    # base单测用例
    │   └── worker  # Worker factory和main Worker（运行IDE Core：Monaco）
    ├── code        # VSCode主运行窗口
    ├── editor        # IDE代码编辑器
    |   ├── browser     # 代码编辑器核心
    |   ├── common      # 代码编辑器核心
    |   ├── contrib     # vscode 与独立 IDE共享的代码
    |   └── standalone  # 独立 IDE 独有的代码
    ├── platform      # 支持注入服务和平台相关基础服务（文件、剪切板、窗体、状态栏）
    ├── workbench     # 工作区UI布局，功能主界面
    │   ├── api              # 
    │   ├── browser          # 
    │   ├── common           # 
    │   ├── contrib          # 
    │   ├── electron-browser # 
    │   ├── services         # 
    │   └── test             # 
    ├── css.build.js  # 用于插件构建的CSS loader
    ├── css.js        # CSS loader
    ├── editor        # 对接IDE Core（读取编辑/交互状态），提供命令、上下文菜单、hover、snippet等支持
    ├── loader.js     # AMD loader（用于异步加载AMD模块）
    ├── nls.build.js  # 用于插件构建的NLS loader
    └── nls.js        # NLS（National Language Support）多语言loader
```
### 核心层
* base: 提供通用服务和构建用户界面
* platform: 注入服务和基础服务代码
* editor: 微软Monaco编辑器，也可独立运行使用
* wrokbench: 配合Monaco并且给viewlets提供框架：如：浏览器状态栏，菜单栏利用electron实现桌面程序

### 核心环境
整个项目完全使用typescript实现，electron中运行主进程和渲染进程，使用的api有所不同，所以在core中每个目录组织也是按照使用的api来安排，
运行的环境分为几类：
* common: 只使用javascritp api的代码，能在任何环境下运行
* browser: 浏览器api, 如操作dom; 可以调用common
* node: 需要使用node的api,比如文件io操作
* electron-brower: 渲染进程api, 可以调用common, brower, node, 依赖[electron renderer-process API](https://github.com/electron/electron/tree/master/docs#modules-for-the-renderer-process-web-page)
* electron-main: 主进程api, 可以调用: common, node 依赖于[electron main-process AP](https://github.com/electron/electron/tree/master/docs#modules-for-the-main-process)

![img](https://github.com/fzxa/VSCode-sourcecode-analysis/blob/master/vscode-welcome.png)


## <a name="3">启动主进程</a>

### Electron通过package.json中的main字段来定义应用入口。
main.js是vscode的入口。

* src/main.js
	* vs/code/electron-main/main.ts
	* vs/code/electron-main/app.ts
	* vs/code/electron-main/windows.ts
	* vs/workbench/electron-browser/desktop.main.ts
	* vs/workbench/browser/workbench.ts

```js
app.once('ready', function () {
	//启动追踪，后面会讲到，跟性能检测优化相关。
	if (args['trace']) {
		// @ts-ignore
		const contentTracing = require('electron').contentTracing;

		const traceOptions = {
			categoryFilter: args['trace-category-filter'] || '*',
			traceOptions: args['trace-options'] || 'record-until-full,enable-sampling'
		};

		contentTracing.startRecording(traceOptions, () => onReady());
	} else {
		onReady();
	}
});
function onReady() {
	perf.mark('main:appReady');

	Promise.all([nodeCachedDataDir.ensureExists(), userDefinedLocale]).then(([cachedDataDir, locale]) => {
		//1. 这里尝试获取本地配置信息，如果有的话会传递到startup
		if (locale && !nlsConfiguration) {
			nlsConfiguration = lp.getNLSConfiguration(product.commit, userDataPath, metaDataFile, locale);
		}

		if (!nlsConfiguration) {
			nlsConfiguration = Promise.resolve(undefined);
		}

		
		nlsConfiguration.then(nlsConfig => {
			
			//4. 首先会检查用户语言环境配置，如果没有设置默认使用英语 
			const startup = nlsConfig => {
				nlsConfig._languagePackSupport = true;
				process.env['VSCODE_NLS_CONFIG'] = JSON.stringify(nlsConfig);
				process.env['VSCODE_NODE_CACHED_DATA_DIR'] = cachedDataDir || '';

				perf.mark('willLoadMainBundle');
				//使用微软的loader组件加载electron-main/main文件
				require('./bootstrap-amd').load('vs/code/electron-main/main', () => {
					perf.mark('didLoadMainBundle');
				});
			};

			// 2. 接收到有效的配置传入是其生效，调用startup
			if (nlsConfig) {
				startup(nlsConfig);
			}

			// 3. 这里尝试使用本地的应用程序
			// 应用程序设置区域在ready事件后才有效
			else {
				let appLocale = app.getLocale();
				if (!appLocale) {
					startup({ locale: 'en', availableLanguages: {} });
				} else {

					// 配置兼容大小写敏感，所以统一转换成小写
					appLocale = appLocale.toLowerCase();
					// 这里就会调用config服务，把本地配置加载进来再调用startup
					lp.getNLSConfiguration(product.commit, userDataPath, metaDataFile, appLocale).then(nlsConfig => {
						if (!nlsConfig) {
							nlsConfig = { locale: appLocale, availableLanguages: {} };
						}

						startup(nlsConfig);
					});
				}
			}
		});
	}, console.error);
}
```

### vs/code/electron-main/main.ts
electron-main/main 是程序真正启动的入口,进入main process初始化流程.
#### 这里主要做了两件事情：
1. 初始化Service 
2. 启动主实例

直接看startup方法的实现,基础服务初始化完成后会加载 CodeApplication, mainIpcServer, instanceEnvironment，调用 startup 方法启动APP
```js
private async startup(args: ParsedArgs): Promise<void> {

		//spdlog 日志服务
		const bufferLogService = new BufferLogService();
		
		// 1. 调用 createServices
		const [instantiationService, instanceEnvironment] = this.createServices(args, bufferLogService);
		try {

			// 1.1 初始化Service服务
			await instantiationService.invokeFunction(async accessor => {
				// 基础服务，包括一些用户数据，缓存目录
				const environmentService = accessor.get(IEnvironmentService);
				// 配置服务
				const configurationService = accessor.get(IConfigurationService);
				// 持久化数据
				const stateService = accessor.get(IStateService);

				try {
					await this.initServices(environmentService, configurationService as ConfigurationService, stateService as StateService);
				} catch (error) {

					// 抛出错误对话框
					this.handleStartupDataDirError(environmentService, error);

					throw error;
				}
			});

			// 1.2 启动实例
			await instantiationService.invokeFunction(async accessor => {
				const environmentService = accessor.get(IEnvironmentService);
				const logService = accessor.get(ILogService);
				const lifecycleService = accessor.get(ILifecycleService);
				const configurationService = accessor.get(IConfigurationService);

				const mainIpcServer = await this.doStartup(logService, environmentService, lifecycleService, instantiationService, true);

				bufferLogService.logger = new SpdLogService('main', environmentService.logsPath, bufferLogService.getLevel());
				once(lifecycleService.onWillShutdown)(() => (configurationService as ConfigurationService).dispose());
				
				return instantiationService.createInstance(CodeApplication, mainIpcServer, instanceEnvironment).startup();
			});
		} catch (error) {
			instantiationService.invokeFunction(this.quit, error);
		}
	}
```


#### vs/code/electron-main/app.ts
这里首先触发CodeApplication.startup()方法， 在第一个窗口打开3秒后成为共享进程，
```js
async startup(): Promise<void> {
	...

	// 1. 第一个窗口创建共享进程
	const sharedProcess = this.instantiationService.createInstance(SharedProcess, machineId, this.userEnv);
	const sharedProcessClient = sharedProcess.whenReady().then(() => connect(this.environmentService.sharedIPCHandle, 'main'));
	this.lifecycleService.when(LifecycleMainPhase.AfterWindowOpen).then(() => {
		this._register(new RunOnceScheduler(async () => {
			const userEnv = await getShellEnvironment(this.logService, this.environmentService);

			sharedProcess.spawn(userEnv);
		}, 3000)).schedule();
	});
	// 2. 创建app实例
	const appInstantiationService = await this.createServices(machineId, trueMachineId, sharedProcess, sharedProcessClient);


	// 3. 打开一个窗口 调用 
	
	const windows = appInstantiationService.invokeFunction(accessor => this.openFirstWindow(accessor, electronIpcServer, sharedProcessClient));

	// 4. 窗口打开后执行生命周期和授权操作
	this.afterWindowOpen();
	...

	//vscode结束了性能问题的追踪
	if (this.environmentService.args.trace) {
		this.stopTracingEventually(windows);
	}
}
```


openFirstWindow 主要实现
CodeApplication.openFirstWindow 首次开启窗口时，创建 Electron 的 IPC，使主进程和渲染进程间通信。
window会被注册到sharedProcessClient，主进程和共享进程通信
根据 environmentService 提供的参数(path,uri)调用windowsMainService.open 方法打开窗口
```js
private openFirstWindow(accessor: ServicesAccessor, electronIpcServer: ElectronIPCServer, sharedProcessClient: Promise<Client<string>>): ICodeWindow[] {

		...
		// 1. 注入Electron IPC Service, windows窗口管理，菜单栏等服务

		// 2. 根据environmentService进行参数配置
		const macOpenFiles: string[] = (<any>global).macOpenFiles;
		const context = !!process.env['VSCODE_CLI'] ? OpenContext.CLI : OpenContext.DESKTOP;
		const hasCliArgs = hasArgs(args._);
		const hasFolderURIs = hasArgs(args['folder-uri']);
		const hasFileURIs = hasArgs(args['file-uri']);
		const noRecentEntry = args['skip-add-to-recently-opened'] === true;
		const waitMarkerFileURI = args.wait && args.waitMarkerFilePath ? URI.file(args.waitMarkerFilePath) : undefined;

		...

		// 打开主窗口，默认从执行命令行中读取参数 
		return windowsMainService.open({
			context,
			cli: args,
			forceNewWindow: args['new-window'] || (!hasCliArgs && args['unity-launch']),
			diffMode: args.diff,
			noRecentEntry,
			waitMarkerFileURI,
			gotoLineMode: args.goto,
			initialStartup: true
		});
	}
```



#### vs/code/electron-main/windows.ts
接下来到了electron的windows窗口，open方法在doOpen中执行窗口配置初始化，最终调用openInBrowserWindow -> 执行doOpenInBrowserWindow是其打开window,主要步骤如下：
```js
private openInBrowserWindow(options: IOpenBrowserWindowOptions): ICodeWindow {

	...
	// New window
	if (!window) {
		//1.判断是否全屏创建窗口
		 ...
		// 2. 创建实例窗口
		window = this.instantiationService.createInstance(CodeWindow, {
			state,
			extensionDevelopmentPath: configuration.extensionDevelopmentPath,
			isExtensionTestHost: !!configuration.extensionTestsPath
		});

		// 3.添加到当前窗口控制器
		WindowsManager.WINDOWS.push(window);

		// 4.窗口监听器
		window.win.webContents.removeAllListeners('devtools-reload-page'); // remove built in listener so we can handle this on our own
		window.win.webContents.on('devtools-reload-page', () => this.reload(window!));
		window.win.webContents.on('crashed', () => this.onWindowError(window!, WindowError.CRASHED));
		window.win.on('unresponsive', () => this.onWindowError(window!, WindowError.UNRESPONSIVE));
		window.win.on('closed', () => this.onWindowClosed(window!));

		// 5.注册窗口生命周期
		(this.lifecycleService as LifecycleService).registerWindow(window);
	}

	...
	
	return window;
}
```
doOpenInBrowserWindow会调用window.load方法 在window.ts中实现
```js
load(config: IWindowConfiguration, isReload?: boolean, disableExtensions?: boolean): void {
	
	...

	// Load URL
	perf.mark('main:loadWindow');
	this._win.loadURL(this.getUrl(configuration));

	...
}

private getUrl(windowConfiguration: IWindowConfiguration): string {

	...
	//加载欢迎屏幕的html
	let configUrl = this.doGetUrl(config);
	...
	return configUrl;
}

//默认加载 vs/code/electron-browser/workbench/workbench.html
private doGetUrl(config: object): string {
	return `${require.toUrl('vs/code/electron-browser/workbench/workbench.html')}?config=${encodeURIComponent(JSON.stringify(config))}`;
}
```
main process的使命完成, 主界面进行构建布局。


在workbench.html中加载了workbench.js，
这里调用return require('vs/workbench/electron-browser/desktop.main').main(configuration);实现对主界面的展示


#### vs/workbench/electron-browser/desktop.main.ts
创建工作区，调用workbench.startup()方法，构建主界面展示布局
```js
...
async open(): Promise<void> {
	const services = await this.initServices();
	await domContentLoaded();
	mark('willStartWorkbench');

	// 1.创建工作区
	const workbench = new Workbench(document.body, services.serviceCollection, services.logService);

	// 2.监听窗口变化
	this._register(addDisposableListener(window, EventType.RESIZE, e => this.onWindowResize(e, true, workbench)));

	// 3.工作台生命周期
	this._register(workbench.onShutdown(() => this.dispose()));
	this._register(workbench.onWillShutdown(event => event.join(services.storageService.close())));

	// 3.启动工作区
	const instantiationService = workbench.startup();

	...
}
...
```

#### vs/workbench/browser/workbench.ts
工作区继承自layout类，主要作用是构建工作区，创建界面布局。
```js
export class Workbench extends Layout {
	...
	startup(): IInstantiationService {
		try {
			...
			
			// Services
			const instantiationService = this.initServices(this.serviceCollection);

			instantiationService.invokeFunction(async accessor => {
				const lifecycleService = accessor.get(ILifecycleService);
				const storageService = accessor.get(IStorageService);
				const configurationService = accessor.get(IConfigurationService);

				// Layout
				this.initLayout(accessor);

				// Registries
				this.startRegistries(accessor);

				// Context Keys
				this._register(instantiationService.createInstance(WorkbenchContextKeysHandler));

				// 注册监听事件
				this.registerListeners(lifecycleService, storageService, configurationService);

				// 渲染工作区
				this.renderWorkbench(instantiationService, accessor.get(INotificationService) as NotificationService, storageService, configurationService);

				// 创建工作区布局
				this.createWorkbenchLayout(instantiationService);

				// 布局构建
				this.layout();

				// Restore
				try {
					await this.restoreWorkbench(accessor.get(IEditorService), accessor.get(IEditorGroupsService), accessor.get(IViewletService), accessor.get(IPanelService), accessor.get(ILogService), lifecycleService);
				} catch (error) {
					onUnexpectedError(error);
				}
			});

			return instantiationService;
		} catch (error) {
			onUnexpectedError(error);

			throw error; // rethrow because this is a critical issue we cannot handle properly here
		}
	}
	...
}
```
