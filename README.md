lua-lluv
========
[![Licence](http://img.shields.io/badge/Licence-MIT-brightgreen.svg)](LICENSE)
[![Build Status](https://travis-ci.org/moteus/lua-lluv.svg?branch=master)](https://travis-ci.org/moteus/lua-lluv)

##Lua low level binding to [libuv](https://github.com/libuv/libuv)

## Compatible with libuv>=1.0.0

##Install

Current master
```
luarocks install lluv --server=https://rocks.moonscript.org/dev
```

### Example

Basic tcp/pipe echo server
```Lua
local uv = require "lluv"

local function on_write(cli, err)
  if err then return cli:close() end
end

local function on_read(cli, err, data)
  if err then return cli:close() end
  cli:write(data, on_write)
end

local function on_connection(server, err)
  if err then return server:close() end
  server
    :accept()
    :start_read(on_read)
end

local function on_bind(server, err, host, port)
  if err then
    print("Bind fail:" .. tostring(err))
    return server:close()
  end

  if port then host = host .. ":" .. port end
  print("Bind on: " .. host)

  server:listen(on_connection)
end

uv.tcp():bind("127.0.0.1", 5555, on_bind)

uv.pipe():bind([[\\.\pipe\sock.echo]], on_bind)

uv.run()
```

Coroutine based echo server
```Lua
local uv     = require "lluv"
local ut     = require "lluv.utils"
local socket = require "lluv.luasocket"

local function echo(cli)
  while true do
    local msg, err = cli:receive("*r")
    if not msg then break end
    cli:send(msg)
  end
  cli:close()
end

local function server(host, port, fn)
  local srv = socket.tcp()

  srv:bind(host, port)

  while true do
    local cli = srv:accept()

    ut.corun(function()
      cli:attach() -- attach socket to current coroutine
      fn(cli)
    end)
  end
end

ut.corun(server, "127.0.0.1", 5555, echo)

uv.run()
```