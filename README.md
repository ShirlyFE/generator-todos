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


## Yeoman三剑客：构建(Grunt,Gulp)工具、包管理工具(Bower, npm)、脚手架工具(Yo)

YO是yeoman的核心工具，也就是项目生产环境和编译环境生成工具，Bower是Web开发的包管理器，概念上类似npm，npm专注于nodeJs模块，而bower专注于CSS、JavaScript、图像等前端相关内容的管理。可以说：效率和规范是yeoman的核心诉求，旨在为开发者提供一系列健壮的工具、程序库和工作流


目前与Grunt构建工具分庭抗礼的是Gulp,而Gulp的目标是取代Grunt

扯些别的，在generator-avalon中我使用了browserify完成对oniui的打包工作，在我看来它的配置比requirejs要方便的多，无怪乎有人说：

> Grunt and RequireJS are out, it’s all about Gulp and Browserify now

Grunt配置文件Gruntfile.js主要做两件事：配置task(文件来源、目的地、选项)；注册task(组合要执行的tasks).使用过程中存在的问题：糟糕的流程控制(task必须先读取文件，修改完成之后将结果写到磁盘文件才可供下个task读取处理，频繁的磁盘IO造成构建效率低下，插件间无法有效串联)；配置项过多,配置规则不尽相同，学习成本自然上升；维护度和清度上也略差。

为了解决Grunt使用中遇到的各种诸如上述问题，Gulp应运而生！Gulp is a streaming build system. 基于Node的stream机制(Node中文件访问、输入输出、HTTP连接等都是stream)，从stream中读取输入做些处理再输出到stream中(类似管道概念)，使构建更高效；代码优于配置的策略使配置更简单，复杂的任务变得可管理；尽量少的API(核心API只有5个)减少学习成本。

不过需要注意的是，gulp还相对年轻，而Grunt有完善的社区，插件丰富。选择哪种构建工具还是需要理智对待

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

## API description

generator的制作是基于generator-generator模板，此模板中引用了lodash、underscore.string、yeoman-generator、chalk、yosay等模块，所以制作自己的generator时可以通过熟悉以上module的api充分的利用generator-generator为自己提供的资源

在generator-generator的index.js中有段代码：

```javascript
askForGeneratorName: function () {
    var prompts = [{
        name: 'generatorName',
        message: 'What\'s the base name of your generator?',
        default: generatorName
    }, {
        type: 'confirm',
        name: 'askNameAgain',
        message: 'The name above already exists on npm, choose another?',
        default: true,
        when: function (answers) {  //这里根据上一个问题的answer来决定是否提出此question，when返回true则执行，false不执行
            var done = this.async();
            var name = 'generator-' + answers.generatorName;
            npmName(name, function (err, available) {
                if (!available) {
                  done(true);
                }

                done(false);
            });
        }
    }];
    this.prompt(prompts, function (props) {
        if (props.askNameAgain) {
            return this.prompting.askForGeneratorName.call(this); //重新提问
        }
        this.generatorName = props.generatorName;
        this.appname = 'generator-' + this.generatorName;
        done();
    }.bind(this));
}
```

generator-generator最终依赖的是yeoman-generator，以下对yeoman-generator的使用做点小说明。

yeoman-generator依赖于fs、util、path、events、assert、lodash、async、findup-sync、nopt、file-utils、through2、user-home、util/engines、util/conflicter、utl/storage、util/prompt-suggestion、class-extend模块，所以制作过程中有任何疑问可以直接去了解这些module的api

```javascript
 /* The `Base` class provides the common API shared by all generators.
 * It define options, arguments, hooks, file, prompt, log, API, etc.
 *
 * It mixes on its prototype all methods you'll find in the `actions/` mixins.
 *
 * Every generator should extend this base class.
 *
 * @constructor
 * @mixes util/common
 * @mixes actions/actions
 * @mixes actions/fetch
 * @mixes actions/file
 * @mixes actions/install
 * @mixes actions/invoke
 * @mixes actions/spawn_command
 * @mixes actions/string
 * @mixes actions/remote
 * @mixes actions/user
 * @mixes actions/wiring
 * @mixes actions/help
 * @mixes nodejs/EventEmitter
 *
    _.extend(Base.prototype, require('./actions/actions'));
    _.extend(Base.prototype, require('./actions/fetch'));
    _.extend(Base.prototype, require('./actions/file'));
    _.extend(Base.prototype, require('./actions/install'));
    _.extend(Base.prototype, require('./actions/string'));
    _.extend(Base.prototype, require('./actions/remote'));
    _.extend(Base.prototype, require('./actions/wiring'));
    _.extend(Base.prototype, require('./actions/help'));
    _.extend(Base.prototype, require('./util/common'));
    ......
 * @param {String|Array} args
 * @param {Object} options
 *
 * @property {Object}   env         - the current Environment being run
 * @property {Object}   args        - Provide arguments at initialization
 * @property {String}   resolved    - the path to the current generator
 * @property {String}   description - Used in `--help` output
 * @property {String}   appname     - The application name
 * @property {Storage}  config      - `.yo-rc` config file manager
 * @property {Object}   fs          - An instance of {@link https://github.com/SBoudrias/mem-fs-editor Mem-fs-editor}
 * @property {Function} log         - Output content through Interface Adapter
 *
 * @example
 * var generator = require('yeoman-generator');
 * var MyGenerator = generator.Base.extend({
 *   writing: function() {
 *     this.fs.write('var foo = 1;', this.destinationPath('index.js'));
 *   }
 * });
 */

 // 每个generator对象的options对象包含的有用信息：
 options: {
    env: { // 当前generator的执行环境
        cwd: 'c:\\Users\lili.tian\Desktop\demo\todosProject'，
        runLoop: {
            queueName: [],
            running: true
        },
        lookups: ['.', 'generator', 'lib/generator'],
        arguments: [],
        options: {}
    },
    resolved: 'c:\\Users\...\generator-todos\app\index.js',
    namespace: 'todos:app'
 }
```
在generator的initializing方法中，其收到的参数是参数组成的对象，比如执行命令：
```
yo todos sync --coffee
```
在initializing方法中this.log(arguments)打印的结果是：{0: sync},所以要获取命令行参数还是需要this.args(this.arguments)来得到我们预期的结果['aaa'],至于选项coffee可以通过this.options.coffee来确定我们要求使用coffee选项，当然只写--coffee默认得到coffee的值为true，那如果我们要给选项赋值，就需要___yo todos --test-framework=jasmin___ 这样this.options['test-framework']的值就是jasmin而不是true了

### 每个generator对象都会有的关键属性的实现细节：
```javascript
{
    appname: (function determineAppname() {
        var appname;
        try {
            appname = require(path.join(process.cwd(), 'bower.json')).name;
        } catch (e) {
        try {
            appname = require(path.join(process.cwd(), 'package.json')).name;
        } catch (e) {}
        }

        if (!appname) {
            appname = path.basename(process.cwd());
        }

        return appname.replace(/[^\w\s]+?/g, ' ');
    })(),
    async: function() {
        running = true;
        return function(err) {
            if (err) self.emit('error', err);
            completed();
        };
    },
    src: [file-utils].createEnv({base: '', dest: '', logger: _.noop}),
    dest: [file-utils].createEnv({base: '', dest: '', logger: _.noop}),
    fs: [mem-fs-editor].create(this.env.sharedFs)
}
```

```javascript
process.chdir(dirPath) --- 改变当前执行目录为dirPath
```

### gruntfile的读取操作

```javascript
Object.defineProperty(this, 'gruntfile', {
    get: function () {
      var gruntfile;
      if (!this.env.gruntfile) {
        // Use actual Gruntfile.js or the default one
        try {
          gruntfile = this.dest.read('Gruntfile.js');
        } catch (e) {}
        this.env.gruntfile = new GruntfileEditor(gruntfile);
      }

      // Schedule the creation/update of the Gruntfile
      this.env.runLoop.add('writing', function (done) {
        this.dest.write('Gruntfile.js', this.env.gruntfile.toString());
        done();
      }.bind(this), { once: 'gruntfile:write' });

      return this.env.gruntfile;
    }
});
```
### hookFor
```javascript
/**
 * Registers a hook to invoke when this generator runs.
 *
 * A generator with a namespace based on the value supplied by the user
 * to the given option named `name`. An option is created when this method is
 * invoked and you can set a hash to customize it.
 *
 * Must be called prior to the generator run (shouldn't be called within
 * a generator "step" - top-level methods).
 *
 * ### Options:
 *
 *   - `as`      The context value to use when runing the hooked generator
 *   - `args`    The array of positional arguments to init and run the generator with
 *   - `options` An object containing a nested `options` property with the hash of options
 *               to use to init and run the generator with
 *
 * ### Examples:
 *
 *     // $ yo webapp --test-framework jasmine
 *     this.hookFor('test-framework');
 *     // => registers the `jasmine` hook
 *
 *     // $ yo mygen:subgen --myargument
 *     this.hookFor('mygen', {
 *       as: 'subgen',
 *       options: {
 *         options: {
 *           'myargument': true
 *         }
 *       }
 *     }
 *
 * @param {String} name
 * @param {Object} config
 */

Base.prototype.hookFor = function hookFor(name, config) {
  config = config || {};

  // enforce use of hookFor during instantiation
  assert(!this._running, 'hookFor can only be used inside the constructor function');

  // add the corresponding option to this class, so that we output these hooks
  // in help
  this.option(name, {
    desc: this._.humanize(name) + ' to be invoked',
    defaults: this.options[name] || ''
  });

  this._hooks.push(_.defaults(config, {
    name: name
  }));

  return this;
};
```
调用hookFor方法必须将其放在
constructor方法中执行才有效。并且hookFor会自动将args带过去，当然如果显示的定义了args则使用显示定义的args初始化generator hook。而composeWith则不会自动将args带过去，所以compose的generator需要的参数或者选项只能显示的定义在options上


### composeWith

```javascript
/**
 * Compose this generator with another one.
 * @param  {String} namespace  The generator namespace to compose with
 * @param  {Object} options    The options passed to the Generator
 * @param  {Object} [settings] Settings hash on the composition relation
 * @param  {string} [settings.local]        Path to a locally stored generator
 * @param  {String} [settings.link="weak"]  If "strong", the composition will occured
 *                                          even when the composition is initialized by
 *                                          the end user
 * @return {this}
 *
 * @example <caption>Using a peerDependency generator</caption>
 * this.composeWith('bootstrap', { options: { sass: true } });
 *
 * @example <caption>Using a direct dependency generator</caption>
 * this.composeWith('bootstrap', { options: { sass: true } }, {
 *   local: require.resolve('generator-bootstrap/app/main.js');
 * });
 */

Base.prototype.composeWith = function composeWith(namespace, options, settings) {
  settings = settings || {};
  var generator;
  if (settings.local) {
    var Generator = require(settings.local);
    Generator.resolved = require.resolve(settings.local);
    Generator.namespace = namespace;
    generator = this.env.instantiate(Generator, options);
  } else {
    generator = this.env.create(namespace, options);
  }

  if (this._running) {
    generator.run();
  } else {
    this._composedWith.push(generator);
  }
  return this;
};
```

在generator的制作中可以使用__dirname，__dirname对应的是当前generator的index.js坐在的目录。比如这里的c:\\users\lili.tian\...\generator-todos\app

## generator制作步骤

1. npm install -g generator-generator
2. 建立一个目录，命名必须为generator-____,下划线是你的generator的真实名字
3. cd到你的generator目录，yo generator命令生成最基本的generator
4. npm link命令使得本地全局可以使用你的generator，当然像使用其他generator一样
5. 修改app下的index.js(generator的入口文件)，和templates下面的内容形成自己想要的generator功能
6. 建立子generator，yo generator:subgenerator "post"命令会创建post/index.js 和post/templates/somefile.js两个文件，你可以像在generator中编辑index.js一样编辑你自己的子generator。然后就可以像这样使用：yo blog:post "hey,buddy"


## yeoman使用遇到问题及解决办法

在用bower安装components时，总是会报错: >>> bower ECMDERR Failed to execute "git ls-remote --tags --heads git://github.com/angular-ui/ui-router.git", exit code of #128
解决方案：change the protocol from git to https, it works >>> git ls-remote --tags --heads https://github.com/angular-ui/ui-router.git
使用此命令实现彻底修改git为https：git config --global url."https://".insteadOf git://



## yeoman 参考资料

[yeoman 基于的文件系统](https://github.com/sboudrias/mem-fs-editor)
[制作generator时所有可用的API](https://travis-ci.org/yeoman/generator/builds/45718180)
[build your own yeoman generator](http://code.tutsplus.com/tutorials/build-your-own-yeoman-generator--cms-20040)


## License

MIT
