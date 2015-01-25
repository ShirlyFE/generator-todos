# generator-todos

> [Yeoman](http://yeoman.io) generator


## Getting Started

### What is Yeoman?

Trick question. It's not a thing. It's this guy:

![](http://i.imgur.com/JHaAlBJ.png)

Basically, he wears a top hat, lives in your computer, and waits for you to tell him what kind of application you wish to create.

Not every new computer comes with a Yeoman pre-installed. He lives in the [npm](https://npmjs.org) package repository. You only have to ask for him once, then he packs up and moves into your hard drive. *Make sure you clean up, he likes new and shiny things.*

```bash
npm install -g yo
```

### Yeoman Generators

Yeoman travels light. He didn't pack any generators when he moved in. You can think of a generator like a plug-in. You get to choose what type of application you wish to create, such as a Backbone application or even a Chrome extension.

To install generator-todos from npm, run:

```bash
npm install -g generator-todos
```

Finally, initiate the generator:

```bash
yo todos
```

### Getting To Know Yeoman

Yeoman has a heart of gold. He's a person with feelings and opinions, but he's very easy to work with. If you think he's too opinionated, he can be easily convinced.

If you'd like to get to know Yeoman better and meet some of his friends, [Grunt](http://gruntjs.com) and [Bower](http://bower.io), check out the complete [Getting Started Guide](https://github.com/yeoman/yeoman/wiki/Getting-Started).


## Yeoman由三剑客构建(Grunt,Gulp)工具、包管理工具(Bower, npm)、脚手架工具(Yo)组成

目前与Grunt构建工具分庭抗礼的是Gulp,而Gulp的目标是取代Grunt

扯些别的，在generator-avalon中我使用了browserify完成对oniui的打包工作，在我看来它的配置比requirejs要方便的多，无怪乎有人说：

> Grunt and RequireJS are out, it’s all about Gulp and Browserify now

Grunt配置文件Gruntfile.js主要做两件事：配置task(文件来源、目的地、选项)；注册task(组合要执行的tasks).使用过程中存在的问题：糟糕的流程控制(task必须先读取文件，修改完成之后将结果写到磁盘文件才可供下个task读取处理，频繁的磁盘IO造成构建效率低下，插件间无法有效串联)；配置项过多,配置规则不尽相同，学习成本自然上升；维护度和清度上也略差。

同样的构建过程，对比一下Grunt和Gulp实现代码：

### Gruntfile.js
```javascript
module.exports = function(grunt) {
    grunt.initConfig({
        concat: {
            'dist/all.js': ['src/*.js']
        },
        uglify: {
            'dist/all.min.js': ['dist/all.js']
        },
        jshint: {
            files: ['gruntfile.js', 'src/*.js']
        },
        watch: {
            files: ['gruntfile.js', 'src/*.js'],
            tasks: ['jshint', 'concat', 'uglify']
        }
    });


    // Load Our Plugins
    grunt.loadNpmTasks('grunt-contrib-jshint');
    grunt.loadNpmTasks('grunt-contrib-concat');
    grunt.loadNpmTasks('grunt-contrib-uglify');
    grunt.loadNpmTasks('grunt-contrib-watch');


    // Register Default Task
    grunt.registerTask('default', ['jshint', 'concat', 'uglify']);
};
```
### Gulpfile.js
```javascript
var gulp = require('gulp');
var jshint = require('gulp-jshint');
var concat = require('gulp-concat');
var rename = require('gulp-rename');
var uglify = require('gulp-uglify');

// Lint JS
gulp.task('lint', function() {
    return gulp.src('src/*.js')
        .pipe(jshint())
        .pipe(jshint.reporter('default'));
});

// Concat & Minify JS
gulp.task('minify', function(){
    return gulp.src('src/*.js')
        .pipe(concat('all.js'))
        .pipe(gulp.dest('dist'))
        .pipe(rename('all.min.js'))
        .pipe(uglify())
        .pipe(gulp.dest('dist'));
});

// Watch Our Files
gulp.task('watch', function() {
    gulp.watch('src/*.js', ['lint', 'minify']);
});

// Default
gulp.task('default', ['lint', 'minify', 'watch']);
```
为了解决Grunt使用中遇到的各种诸如上述问题，Gulp应运而生！Gulp is a streaming build system. 基于Node的stream机制(Node中文件访问、输入输出、HTTP连接等都是stream)，从stream中读取输入做些处理再输出到stream中(类似管道概念)，使构建更高效；代码优于配置的策略使配置更简单，复杂的任务变得可管理；尽量少的API(核心API只有5个)减少学习成本。



不过需要注意的是，gulp还相对年轻，而Grunt有完善的社区，插件丰富。选择哪种构建工具还是需要理智对待

## API description



## License

MIT
