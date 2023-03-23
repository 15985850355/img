# js hook
  
Hook setInterval
-------------------

复制代码`let _setInterval=setInterval;
        setInterval=function(a,b){
            if(a.toString().indexOf("debugger")!=-1){
                return null;
            }
            _setInterval(a,b);
}` 

  
Hook header
--------------

复制代码`var header_old = window.XMLHttpRequest.prototype.setRequestHeader;
window.XMLHttpRequest.prototype.setRequestHeader = function (key, value) {
    if (key=='k'){
        console.log(key, value)
        debugger;
    }
    if (key=='token'){
        console.log(key, value)
        debugger;
    }

    return header_old.apply(this, arguments);
}` 

  
Hook Cookie Info
-------------------

复制代码`var cookie_cache = document.cookie;
        Object.defineProperty(document, 'cookie', {
            get: function() {
                console.log('Getting cookie');
                return cookie_cache;
            },
            set: function(val) {
                console.log('Setting cookie', val);
                var cookie = val.split(";")[0];
                var ncookie = cookie.split("=");
                var flag = false;
                var cache = cookie_cache.split("; ");
                cache = cache.map(function(a){
                    if (a.split("=")[0] === ncookie[0]){
                        flag = true;
                        return cookie;
                    }
                    return a;
                })
                cookie_cache = cache.join("; ");
                if (!flag){
                    cookie_cache += cookie + "; ";
                }
                this._value = val;
                return cookie_cache;
            },
        });` 

  
Hook Json Info
-----------------

复制代码`var my_stringify = JSON.stringify;
        JSON.stringify = function (params){
            console.log("json_stringify:", params);
            return json_stringify(params);
        };

        var my_parse = JSON.parse;
        JSON.parse = function (params){
            console.log("json_parse:", params);
            return json_parse(params);
        };` 

  
Hook WebSocket Info
----------------------

复制代码`WebSocket.prototype.senda = WebSocket.prototype.send;
        WebSocket.prototype.send = function (data){
         console.info("Hook WebSocket", data);
         return this.senda(data)
        }` 

  
Hook Cookie
--------------

复制代码`(function() {
    'use strict';
    var cookie_cache = document.cookie;
    Object.defineProperty(document, 'cookie', {
        get: function() {
            
            return cookie_cache;
        },
        set: function(val) {
            if (val.indexOf('gdxidpyhxdE') != -1){
                console.log('cookie',val)
                debugger;
            }
            var cookie = val.split(";")[0];
            var ncookie = cookie.split("=");
            var flag = false;
            var cache = cookie_cache.split(";");
            cache = cache.map(function(a){
                if (a.split("=")[0] === ncookie[0]){
                    flag = true;
                    return cookie;
                }
                return a;
            })
            cookie_cache = cache.join(";");
            if (!flag){
                cookie_cache += cookie + ";";
            }
        },
    });

})();` 

  
Hook XHR
-----------

复制代码 `function hookFunction(funcName, config) {
  return function () {
    var args = Array.prototype.slice.call(arguments)
    
    if (funcName === 'open') {
      this.xhr.open_args = args
    }
    if (config[funcName]) {
      console.log(this, 'this')
      
      var result = config[funcName].call(this, args, this.xhr)
      if (result) return result;
    }
    return this.xhr[funcName].apply(this.xhr, arguments);
  }
}

function getterFactory(attr, config) {
  return function () {
    var value = this.hasOwnProperty(attr + "_") ? this[attr + "_"] : this.xhr[attr];
    var getterHook = (config[attr] || {})["getter"]
    return getterHook && getterHook(value, this) || value
  }
}

function setterFactory(attr, config) {
  return function (value) {
    var _this = this;
    var xhr = this.xhr;
    var hook = config[attr]; 
    this[attr + "_"] = value;
    if (/^on/.test(attr)) {
      
      xhr[attr] = function (e) {
        
        var result = hook && config[attr].call(_this, xhr, e)
        result || value.call(_this, e);
      }
    } else {
      var attrSetterHook = (hook || {})["setter"]
      value = attrSetterHook && attrSetterHook(value, _this) || value
      try {
        
        xhr[attr] = value;
      } catch (e) {
        console.warn('xhr的' + attr + '属性不可写')
      }
    }
  }
}

function xhrHook(config) {
  
  window.realXhr = window.realXhr || XMLHttpRequest
  
  XMLHttpRequest = function () {
    var xhr = new window.realXhr()
    
    this.xhr = xhr
    
    for (var attr in xhr) {
      if (Object.prototype.toString.call(xhr[attr]) === '[object Function]') {
        this[attr] = hookFunction(attr, config); 
      } else {
        
        Object.defineProperty(this, attr, { 
          get: getterFactory(attr, config),
          set: setterFactory(attr, config),
          enumerable: true
        })
      }
    }
  }
  return window.realXhr
}

function unXhrHook() {
  if (window[realXhr]) XMLHttpRequest = window[realXhr];
  window[realXhr] = undefined;
}

xhrHook({
  open: function (args, xhr) {
    console.log("open called!", args, xhr)
     
  },
  setRequestHeader: function (args, xhr) {
    console.log("setRequestHeader called!", args, xhr)
         },
  onload: function (xhr) {
    
    this.responseText += ' tager'
  }
})` 

Hook debugger
-------------

复制代码 `Function.prototype.temp_constructor= Function.prototype.constructor;
Function.prototype.constructor=function(){
    if (arguments && typeof arguments[0]==="string"){
        if (arguments[0]==="debugger")
        return ""
    }
    return Function.prototype.temp_constructor.apply(this, arguments);
};

_setInterval = setInterval
setInterval = function setInterval(code, time){
    console.log(code, time)
    code = code.toString().replace(/debugger/, "").replace(/function ()/, "function xxx")
    return _setInterval(new Function(code) , time)
}
_setTimeout = setTimeout
setTimeout = function setTimeout(code, time){
    console.log(code, time)
    code = code.toString().replace(/debugger/, "").replace(/function ()/, "function xxx")
    return _setTimeout(new Function(code), time)
}`