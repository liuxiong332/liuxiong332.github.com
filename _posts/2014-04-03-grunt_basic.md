---
layout: post
title:  "Grunt!"
categories: node
---

## Grunt是什么

Grunt是nodejs用来自动化的工具。它就像Make一样，能将压缩(minification)，编译，测试等动作进行自动化。就像Make中的目标一样，Grunt将每一个动作称作一个任务。一个工程有许多任务组成。

## Grunt任务

Makefile形如如下所示来定义目标：

```sh
bin: 
	gcc main.cc
```

类似的，Grunt像如下来定义任务：

```js
module.exports = function(grunt) {

  // A very basic default task.
  grunt.registerTask('default', 'Log some stuff.', function() {
    grunt.log.write('Logging some stuff...').ok();
  });

};
```

其中任务名字是default，任务描述是'Log some stuff'，任务内容是registerTask的第三个参数function。
Grunt将任务定义在Gruntfile.js中，Gruntfile是内容形如上面代码的nodejs文件：

对于压缩、编译和单元测试这些任务，Grunt定义了一些插件来支持这些任务。因此我们只需要在package.json文件的devDependencies中引用这些插件名字，然后调用`npm install`来安装这些插件即可使用。

```js
{
  "name": "my-project-name",
  "version": "0.1.0",
  "devDependencies": {
    "grunt": "~0.4.2",
    "grunt-contrib-jshint": "~0.6.3",
    "grunt-contrib-nodeunit": "~0.2.0",
    "grunt-contrib-uglify": "~0.2.2"
  }
}
```

上述package引用了jshint、nodeunit、uglify这些Grunt插件。安装了这些Grunt插件，我们需要在Gruntfile中调用`loadNpmTask`来加载这些任务。

```js
// Load the plugin that provides the "uglify" task.
grunt.loadNpmTasks('grunt-contrib-uglify');

// Default task(s).
grunt.registerTask('default', ['uglify']);
```

上述代码中加载了uglify任务，它用于压缩css和javascript文件。自定义的任务`default`就调用了uglify任务。default任务是当使用Grunt或者Grunt default时调用的任务。

## Grunt任务配置

这些任务都需要配置一些参数，例如任务的源文件和目标文件。Grunt使用下面的方式来为任务配置参数：

```js
// Project configuration.
grunt.initConfig({
  uglify: {
    options: {
      banner: '\n'
    },
    build: {
      src: 'src/*.js',
      dest: 'build/*.min.js'
    }
  }
});
```

通过调用`initConfig`来初始化任务参数。参数的属性为任务名称，例如uglify是grunt-contrib-uglify(去掉grunt-contrib前缀)任务的名称，options是uglify任务的参数，options中的banner属性用来配置uglify显示的标题。uglify中有个build又是什么呢？Grunt可以在任务中定义目标。build就是uglify的一个目标，build中的参数是目标需要的参数，对于这些目标Grunt可以使用 uglify:build 来调用。

```js
grunt.initConfig({
  concat: {
    foo: {
      // concat task "foo" target options and files go here.
    },
    bar: {
      // concat task "bar" target options and files go here.
    },
  },
  uglify: {
    bar: {
      // uglify task "bar" target options and files go here.
    },
  },
});
```

可以像上面一样定义任务中的目标，通过`Grunt task:target`来调用目标。例如Grunt concat:foo来调用foo目标任务。