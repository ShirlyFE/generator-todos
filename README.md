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


## Yeoman由三剑客Grunt、Bower、Yo组成

目前与Grunt构建工具分庭抗礼的是Gulp

扯些别的，在generator-avalon中我使用了browserify完成对oniui的打包工作，在我看来它的配置比requirejs要方便的多，无怪乎有人说：

> Grunt and RequireJS are out, it’s all about Gulp and Browserify now

Grunt配置文件Gruntfile.js主要做两件事：配置task(文件来源、目的地、选项)；注册task(组合要执行的tasks).使用过程中存在的问题：糟糕的流程控制(task必须先读取文件，修改完成之后将结果写到磁盘文件才可供下个task读取处理，频繁的磁盘IO造成构建效率低下，插件间无法有效串联)；配置项过多,配置规则不尽相同，学习成本自然上升；维护度和清度上也略差。


为了解决Grunt使用中遇到的各种诸如上述问题，Gulp应运而生！Gulp is a streaming build system. 基于Node的stream机制(Node中文件访问、输入输出、HTTP连接等都是stream)，从stream中读取输入做些处理再输出到stream中(类似管道概念)，优雅解决复杂问题，也因此使构建更快速，配置更简单。不过需要注意的是，gulp还相对年轻，选择哪种构建工具还是需要理智对待

## API description



## License

MIT
