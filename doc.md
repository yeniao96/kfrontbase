# 项目说明

## 项目文件结构说明

  ---bin 存放执行kfront-base命令文件的文件

  ---generators 存放kfront-base命令文件相关的逻辑代码和相关模板文件

  ---utils 存放工具脚本

## package.json中相关配置字段说明

  * commands：存放kfront-base可用命令的集合
  * updatePackages: 用户执行kfront-base update 命令后要默认更新的依赖
  * uiComponentsLibs：推荐用户安装的UI组件库的集合，分别针对不同的技术栈做配置

  【注意】尽量将程序中可配置的项放到package.json中，减少在程序中手动修改源码。


## 项目开发注意事项

  * 添加自定义命令的流程

    1.在package.json中的字段“commands”下添加一个选项，例如

    <code>
      <pre>
        {
          "command": "plugin",
          "description": "添加单个插件"
        }
      </pre>
    </code>

    commands下添加的选项字段必须是command和description，command是命令名称，description是命令描述【可通过kfront-base命令查看所以命令及命令描述】。

    2.在package.json中的bin字段下添加执行新命令的入口文件路径，例如

      <code>
        <pre>
          "kfront-base-plugin": "./bin/kfront-base-add-plugin.js"
        </pre>
      </code>

      【命名规范】

        * 必须以“kfront-base-命令名称”的形式命名字段
        * 命令名称中不可包含空格、冒号、斜杠等非法文件名称
        * 字段对应的属性中的文件名称也要和字段名称相同

    3.在package.json中的files字段下添加新命令的入口文件路径和业务逻辑文件路径，例如

      <code>
        <pre>
          "generators/plugin",
          ...
          "bin/kfront-base-add-plugin.js"
        </pre>
      </code>

      字段内容和属性内容和bin字段中添加的相同。

    4.在bin目录下创建执行命令的入口文件，例如 kfront-base-add-plugin.js，内容如下

      <code>
        <pre>
          #!/usr/bin/env node
          var env = require("yeoman-environment").createEnv();
          /**
          * 调用yeoman generator添加单个插件的命令
          */
          env.lookup(function(){
            env.run("kfront-base:plugin");
          });
        </pre>
      </code>

    5.在genarator目录下创建命令相关逻辑代码和模板文件

      【注意事项】

        * 每个命令对应一个文件夹，文件夹名称和命令名称相同
        * 每个文件夹下必须有一个index.js
        * 命令需要创建模板时需要在命令对应的文件夹下创建一个templates文件夹，正对不同的技术栈需要创建相应的文件夹，例如：vue、react等
        * 尽量将模板文件的动态内容写入kfront-base.json中的相关配置中
        * index.js需要遵循yeoman generator-generator 的格式去写

  * 模板文件中配置项的注意事项

    1.模板文件中的简单值示例：

      <code>
        <pre>
          <%= varible %>
        </pre>
      </code>

    2.模板文件中的简单语句示例：

      <code>
        <pre>
          "vuex": "^3.0.1" <% if(pkgOption['element-ui']) { %>,
          "element-ui": "^2.4.4" <% } %>
        </pre>
      </code>

    3.编译模板时一定要添加编译的变量：

     <code>
        <pre>
           /**
             * package.json模板中的选项配置
             */
            var pkgTplOptions = {
              projectName: this.projectName,
              version: this.version,
              pkgOption: this.pkgOption
            };
            /**
             * 编译并复制package.json到用户项目根目录下
             */
            this.fs.copyTpl(
              this.templatePath(filePath),
              this.destinationPath(this.userProjectRootPath + fileName),
              pkgTplOptions
            );
        </pre>
      </code>

  * command-utils.js中函数使用注意事项

    1.executeCommand中参数详细说明

      options：{async: false, slient: false}

        async:是否同步执行命令，默认false

        slient:是否将进程的处理过程显示到控制台，默认false
      
      callback: 回调函数的参数如下

        code:命令执行结果的状态码，0表示成功

        stdout:命令执行的进程过程输出

        stderr:命令执行的进程过程中的错误输出

  * package.json中添加命令注意事项

      将添加的命令名称按字母顺序排序

      <code>
        <pre>
          "commands": [
            {
              "command": "add-plugin",
              "description": "添加单个或多个插件"
            },
            {
              "command": "add-subapp",
              "description": "添加子工程"
            },
            {
              "command": "delete-plugin",
              "description": "删除单个或多个插件"
            },
            {
              "command": "delete-subapp",
              "description": "删除单个或多个子工程"
            },
            {
              "command": "init-app",
              "description": "创建新工程"
            },
            {
              "command": "install",
              "description": "安装项目需要的依赖包"
            },
            {
              "command": "update",
              "description": "更新kfront-base"
            }
          ]
        </pre>
      </code>


## 项目版本控制说明

  1.项目的分支

    * 开发分支：

      feature-stable:稳定版本的功能开发,包括新功能扩展、bug修复等。

      feature-latest:最新版本的功能开发，不同于稳定版的功能扩展，包括技术换型、项目结构重构等。

	  * 发布分支：

      release-stable:稳定版本的功能发布，对应feature-stable。

      release-latest:最新版本的功能发布，对应feature-latest。

    【注意】平时的正常功能的开发在feature-stable上开发，对一些实验性功能的开发可以放在feature-latest上开发。
      NPM发布时要在release-stable上先合并feature-stable再发布。

  2.NPM发布的注意事项

    * NPM版本号数值限制：
    发布版本号目前从2.1.18开始，尽可能减少在NPM仓库的发布，每次发布版本号加1，版本号的最后一个数字控制在20以内（包括20）。



