+++
title = "In-browser Redis"
slug = "redis-in-browser"
date = 2015-01-09
path = "2015/redis-in-browser"
+++

Here is a proof of concept [redis]&nbsp;[emscripten] port I have made. 
It hasn't required a lot of modifications to work, which mostly are commenting out clustering, persistence, lua, sentinel and other stuff.
Maybe the port has other limitations, I haven't done full testing,
but basic stuff works pretty well.
You can try it yourself, autocomplete commands work with the TAB key
(Modern browsers with asm.js support are required for speed).
<br/>
<p>
  <input id="run" type="button" value="Run redis database"/>
</p>
<small id="status">about 700kb scripts will be downloaded</small>

<div class="terminal" id="output" rows="8">
</div>

<footer>
<br/>

[github fork]

[Hacker News post]
</footer>

[emscripten]: http://kripken.github.io/emscripten-site/
[redis]: http://redis.io
[github fork]: https://github.com/narma/redis
[Hacker News post]: https://news.ycombinator.com/item?id=9020245

<style>
  #run {
    background-color: #4CAF50;
    border: none;
    color: white;
    padding: 15px 32px;
    text-align: center;
    text-decoration: none;
    display: inline-block;
    font-size: 16px;
  }
  #run:hover {
    background-color: #3C9F45;
  }
  #run:disabled {
    background-color: gray;
  }
</style>
<script type='text/javascript'>
var statusElement = document.getElementById('status');

function status(text) {
  console.log(text);
  statusElement.innerHTML = text;
}

var Module = {
    preRun: [],
    postRun: [],
    print: (function() {
      //var element = document.getElementById('output');
      //if (element) element.value = ''; // clear browser cache
      return function(text) {
        text = Array.prototype.slice.call(arguments).join(' ');
        console.log(text);
        if (jQuery) {
          jQuery('#output').terminal().echo(text);
        }
      };
    })(),
    printErr: function(text) {
      text = Array.prototype.slice.call(arguments).join(' ');
      if (0) { // XXX disabled for safety typeof dump == 'function') {
        dump(text + '\n'); // fast, straight to the real console
      } else {
        console.error(text);
      }
    },
    setStatus: function(text) {
      if (!Module.setStatus.last) Module.setStatus.last = { time: Date.now(), text: '' };
      if (text === Module.setStatus.text) return;
      var m = text.match(/([^(]+)\((\d+(\.\d+)?)\/(\d+)\)/);
      var now = Date.now();
      statusElement.innerHTML = text;
    },
    totalDependencies: 0
  };
  window.onerror = function(event) {
    Module.setStatus('Exception thrown, see JavaScript console');
    Module.setStatus = function(text) {
      if (text) Module.printErr('[post-exception status] ' + text);
    };
  };
  window.on_redis_ready = function() {
    status('ready');
    jQuery('#output').terminal().focus(true);
  }

function loadScript(url, callback) {

    var script = document.createElement("script")
    script.type = "text/javascript";
    script.async = true;

    if (script.readyState){  //IE
        script.onreadystatechange = function(){
            if (script.readyState == "loaded" ||
                    script.readyState == "complete"){
                script.onreadystatechange = null;
                callback();
            }
        };
    } else {  //Others
        script.onload = callback;
    }

    script.src = url;
    document.getElementsByTagName("head")[0].appendChild(script);
}
function redisloaded() {
  status('Eval redis script...');
}
function parse_result(res, term) {
  var code = res[0];
  if (res == "$-1\r\n") return "(nil)";

  var re = /\$(-|\d)+\s+/gi;

  if (code == "-") return res; // return errors
  if (code == "+") return res.substr(1).trim();
  if (code == "$") {
    return '"' + res.replace(re, "").trim() + '"';
  }
  if (code == ":") {
    return "(integer) " + res.substr(1);
  }
  if (code == "*") {
    var a = res.split('\r\n'),
      i,y = 1, l = a.length, c, result = '';

    for(i = 0; i < l; ++i) {
      c = a[i];
      if (c[0] == '*' || c[0] == '$' || c.length == 0) continue;
      term.echo('' + y + ') "' + c + '"');
      y++;
    }
    return false;
  }

  return res;
}
var cmds = ['APPEND',
 'AUTH',
 'BGREWRITEAOF',
 'BGSAVE',
 'BITCOUNT',
 'BITOP',
 'BITPOS',
 'BLPOP',
 'BRPOP',
 'BRPOPLPUSH',
 'CLIENT GETNAME',
 'CLIENT KILL',
 'CLIENT LIST',
 'CLIENT PAUSE',
 'CLIENT SETNAME',
 'CLUSTER SLOTS',
 'COMMAND',
 'COMMAND COUNT',
 'COMMAND GETKEYS',
 'COMMAND INFO',
 'CONFIG GET',
 'CONFIG RESETSTAT',
 'CONFIG REWRITE',
 'CONFIG SET',
 'DBSIZE',
 'DEBUG OBJECT',
 'DEBUG SEGFAULT',
 'DECR',
 'DECRBY',
 'DEL',
 'DISCARD',
 'DUMP',
 'ECHO',
 'EVAL',
 'EVALSHA',
 'EXEC',
 'EXISTS',
 'EXPIRE',
 'EXPIREAT',
 'FLUSHALL',
 'FLUSHDB',
 'GET',
 'GETBIT',
 'GETRANGE',
 'GETSET',
 'HDEL',
 'HEXISTS',
 'HGET',
 'HGETALL',
 'HINCRBY',
 'HINCRBYFLOAT',
 'HKEYS',
 'HLEN',
 'HMGET',
 'HMSET',
 'HSCAN',
 'HSET',
 'HSETNX',
 'HVALS',
 'INCR',
 'INCRBY',
 'INCRBYFLOAT',
 'INFO',
 'KEYS',
 'LASTSAVE',
 'LINDEX',
 'LINSERT',
 'LLEN',
 'LPOP',
 'LPUSH',
 'LPUSHX',
 'LRANGE',
 'LREM',
 'LSET',
 'LTRIM',
 'MGET',
 'MIGRATE',
 'MONITOR',
 'MOVE',
 'MSET',
 'MSETNX',
 'MULTI',
 'OBJECT',
 'PERSIST',
 'PEXPIRE',
 'PEXPIREAT',
 'PFADD',
 'PFCOUNT',
 'PFMERGE',
 'PING',
 'PSETEX',
 'PSUBSCRIBE',
 'PTTL',
 'PUBLISH',
 'PUBSUB',
 'PUNSUBSCRIBE',
 'QUIT',
 'RANDOMKEY',
 'RENAME',
 'RENAMENX',
 'RESTORE',
 'ROLE',
 'RPOP',
 'RPOPLPUSH',
 'RPUSH',
 'RPUSHX',
 'SADD',
 'SAVE',
 'SCAN',
 'SCARD',
 'SCRIPT EXISTS',
 'SCRIPT FLUSH',
 'SCRIPT KILL',
 'SCRIPT LOAD',
 'SDIFF',
 'SDIFFSTORE',
 'SELECT',
 'SET',
 'SETBIT',
 'SETEX',
 'SETNX',
 'SETRANGE',
 'SHUTDOWN',
 'SINTER',
 'SINTERSTORE',
 'SISMEMBER',
 'SLAVEOF',
 'SLOWLOG',
 'SMEMBERS',
 'SMOVE',
 'SORT',
 'SPOP',
 'SRANDMEMBER',
 'SREM',
 'SSCAN',
 'STRLEN',
 'SUBSCRIBE',
 'SUNION',
 'SUNIONSTORE',
 'SYNC',
 'TIME',
 'TTL',
 'TYPE',
 'UNSUBSCRIBE',
 'UNWATCH',
 'WATCH',
 'ZADD',
 'ZCARD',
 'ZCOUNT',
 'ZINCRBY',
 'ZINTERSTORE',
 'ZLEXCOUNT',
 'ZRANGE',
 'ZRANGEBYLEX',
 'ZRANGEBYSCORE',
 'ZRANK',
 'ZREM',
 'ZREMRANGEBYLEX',
 'ZREMRANGEBYRANK',
 'ZREMRANGEBYSCORE',
 'ZREVRANGE',
 'ZREVRANGEBYLEX',
 'ZREVRANGEBYSCORE',
 'ZREVRANK',
 'ZSCAN',
 'ZSCORE',
 'ZUNIONSTORE'];

function jquery_loaded() {
  status('get jquery.terminal');
  // loadScript('//cdnjs.cloudflare.com/ajax/libs/jquery-mousewheel/3.1.12/jquery.mousewheel.min.js');
  $('<link>')
    .appendTo('head')
    .attr({type : 'text/css', rel : 'stylesheet'})
    .attr('href', '//cdnjs.cloudflare.com/ajax/libs/jquery.terminal/2.21.0/css/jquery.terminal.min.css');

  loadScript('//cdnjs.cloudflare.com/ajax/libs/jquery.terminal/2.21.0/js/jquery.terminal.min.js', function() {
      status('get redis-server');
      $('#output').terminal(function(command, term) {
        if (command !== '') {
            try {
                rsend(command + "\r\n");
                var result = rrecv();
                if (result) {
                  var parsed = parse_result(result, term);
                  if (parsed) term.echo(parsed);
                }
            } catch(e) {
                term.error(new String(e));
            }
        } else {
           term.echo('');
        }
    }, {
        completion: function(text, cb) {
          //debugger;
          var suggests = [], i, cmds_len = cmds.length;
          var t = text.toUpperCase(), current,
            t_len  = text.length;

          var on_screen = this.get_command();
          if (on_screen.length > 0 && on_screen[on_screen.length-1] === ' ') {
            // subcmd search
            cb([]);
            return [];
          } else {
            // cmd search
            for (i=0; i < cmds_len; ++i) {
              current = cmds[i];
              if (current.indexOf(t) == 0) suggests.push(text + current.substr(t_len));
            }
          }
          cb(suggests);
          return suggests;
        },
        greetings: '',
        name: 'redis_js_demo',
        height: 260,
        prompt: 'redis-js> '});
    loadScript('/redis-in-browser/redis-server.js', redisloaded);

  });
}
window.onload = function() {
  statusElement = document.getElementById('status');
  document.getElementById('run').addEventListener('click', function() {
    this.disabled = "disabled";
    status('get jquery');
    loadScript('//cdnjs.cloudflare.com/ajax/libs/jquery/3.4.1/jquery.min.js', jquery_loaded);
  });
}
</script>