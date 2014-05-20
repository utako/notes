# w8 - Node!

## 02 Node Video

### on nodemon package

* Will watch files for us and restart a script if we change one of the files the script uses.

### Note

* Creating a server callback function will act as a request handler:

        var server = http.createServer(function(req, req) {
          // added as a listener for a request event.
        });

## 03 Node Video

Make a simple server in Ruby that gets some input from the client and echo it back out.

### Notes

* This is a TCP server (low level, http is built on this).
* HTTP: client can send data to the server in the request phase, and then the server can respond to the request. Then the server shuts down.
* TCP: Allows bidirectional communication without the server shutting down.

### TCP Servers
* Very simple server that writes the message and closes.

```ruby
require 'socket'
server = TCPServer.new(3000)

while true
  // will block (wait) for a connection to come in
  socket = server.accept
  socket.puts "CONNECTED! Now closing!"
  socket.close
end
```

### Concurrency

* Tell socket server to get some input from the user. Write the input back to the socket.
```ruby
require 'socket'
server = TCPServer.new(3000)

while true
  # will block (wait) for a connection to come in
  socket = server.accept
  input = socket.gets
  sleep(2)
  socket.puts(input)
  sleep(2)
  socket.puts "CLOSING"
  socket.close
end
```
Connecting with netcat will accept input from the console and do what you want it to do.
* The problem with the server as we've written it is that it has low concurrency: the problem is that while it waits for input; it won't accept a second connection until it gets the input.

### Low Concurrency Threads

* Thread: Allows us to run simultaneous lines of code.

```ruby
require 'socket'
require 'thread'

server = TCPServer.new(3000)

while true
  // will block (wait) for a connection to come in
  socket = server.accept
  Thread.new do
    socket.puts "CONNECTED! Now closing!"
    socket.close
  end
end
```
Will spawn more threads to accept more connections. This is a multi-threaded server.
* Cons: New threads can be slow. Can be troublesome in coordinating setting of global variables. The following will return 1000 instead of 10000. WTF
```ruby
$counter = 0
$threads = []
100.times do |i|
  $threads << Thread.new do
    1000.times do
      new_val = $counter + 1
      sleep(0.001)
      $counter = new_val
    end
  end
end
```

### Fixing that with Mutex

* `#synchronize` says, "Use this block and don't allow anyone to use this block until this block is complete." This makes this block "atomic." If mutex is unlocked, you can enter the block. Before we enter the block, we lock the mutex and we get exclusive access to the mutex until we've finished the block. Then, synchronize will unlock the block. No two threads can be inside this code simultaneously.

```ruby
$counter = 0
$threads = []
$mutex = Mutex.new
100.times do |i|
  $threads << Thread.new do
    1000.times do
      mutex.synhronize do
        new_val = $counter + 1
        sleep(0.001)
        $counter = new_val
      end
    end
  end
end
```

### Thread-safe or thread-unsafe
* Thread-safe: Call methods from different threads and not worry about race conditions.
* Thread-unsafe: Calling methods from different threads may corrupt other simultaneous method calls.
* Thread-Unsafe Code:
```ruby
class UnsafeCounter
  attr_reader :count
  def initialize
    @count = 0
  end
  def increment
    new_val = @count + 1
    sleep(0.00001)
    @count = new_val
  end
end

$counter = UnsafeCounter.new
$threads = []
$mutex = Mutex.new
100.times do |i|
  $threads << Thread.new do
    1000.times do
      count.increment
      # mutex.synhronize do
      #   new_val = $counter + 1
      #   sleep(0.001)
      #   $counter = new_val
      # end
    end
  end
end
$threads.each do |thread|
  thread.join
end
puts $counter.count
```

* Thread-Safe Code:
```ruby
class UnsafeCounter
  attr_reader :count
  def initialize
    @count = 0
  end
  def increment
    new_val = @count + 1
    sleep(0.00001)
    @count = new_val
  end
end

class SafeCounter
  def initialize
    @unsafe_counter = UnsafeCounter.new
    @mutex = Mutex.new
  end
  def increment
    @mutex.synchronize do
      @unsafe_counter.increment
    end
  end
  def count
    @mutex.synchronize do
      @unsafe_counter.count
    end
  end
end
$counter = SafeCounter.new
$threads = []
$mutex = Mutex.new
100.times do |i|
  $threads << Thread.new do
    1000.times do
      count.increment
      # mutex.synhronize do
      #   new_val = $counter + 1
      #   sleep(0.001)
      #   $counter = new_val
      # end
    end
  end
end
$threads.each do |thread|
  thread.join
end
puts $counter.count
```
* Pre-emptive threading: Any thread that's running can at any time be stopped and another thread will start running.

## 04 Node Video

* Alternative to thread-based solution to concurrency problem. We'll write a non-blocking ruby server.
```ruby
require 'socket'
$server = TCPServer.new(3000)
$readers = [$server]
def process_ready_readers(ready_readers)
  ready_readers.each do |rr|
    if rr.class == TCPServer
      $readers << rr.accept
    else
      #rr is a TCPSocket
      input = rr.gets
      rr.puts(input)
      rr.close
      $readers.delete(rr)
    end
  end
end
while true
  ready_readers = IO.select($readers)[0]
  process_ready_readers
  # socket = $server.accept
  # input = socket.gets
  # socket.puts(input)
  # socket.close
end
```
* IO.select takes an array of readers. It's going to wait until some (one or many) readers are available to be read.
In our case, it starts out just containing the server. If a reader is returned from IO.select, if we call gets() on it, it will immediately return. The data is already there.
* The first array is a list of readers that are ready to be read.
* IO::select is built into the operating system and is magic.

## 05 Node Views

* Callbacks for asynchronous IO
* If we want to do something different on the second line of input per connection, we need to add a little sophistication to what we're doing.
* The solution is callbacks!
```ruby
require 'socket'
$server = TCPServer.new(3000)
$readers = [$server]
$work = {}

$accept_socket = Proc.new do |server|
  socket = server.accept
  $readers << socket
end

$first_read = Proc.new do |socket|
  input = socket.gets
  socket.puts(input)
  socket.close
  $readers.delete(socket)
  $work.delete(socket)
end

$work[$server] = $accept_socket

while true
  ready_readers = IO.select($readers)[0]
  ready_readers.each do |rr|
    $work[rr].call(rr)
  end
end
```
* Instead of immediately closing the server, let's write a second read.
```ruby
require 'socket'
$server = TCPServer.new(3000)
$readers = [$server]
$work = {}

$accept_socket = Proc.new do |server|
  socket = server.accept
  $work[socket] = $first_read
  $readers << socket
end

$first_read = Proc.new do |socket|
  input = socket.gets
  socket.puts("1": #{input}")
  $work[socket] = $second_read
end

$seond_read = Proc.new do |socket|
  input = socket.gets
  socket.puts("2: #{input}")
  socket.close
  $readers.delete(socket)
  $work.delete(socket)
end

$work[$server] = $accept_socket

while true
  ready_readers = IO.select($readers)[0]
  ready_readers.each do |rr|
    $work[rr].call(rr)
  end
end
```
* Next, make an async_read method that takes the proc and saves the work in the hash. This is a callback in ruby. async_read takes the callback, and adds it to the list of readers, and then runs the callback when it's ready to read.
```ruby
require 'socket'
$server = TCPServer.new(3000)
$readers = [$server]
$work = {}

def async_read(socket, &prc)
  $readers << socket
  $work[socket] = Proc.new do
    $readers.delete(socket)
    $work.delete(socket)
    prc.call
  end
end

$accept_socket = Proc.new do |server|
  socket = server.accept
  async_read(socket) do
    input1 = socket.gets
    socket.puts("1: #{input1}")
    async_read(socket) do
      input2 = socket.gets
      socket.puts("1: #{input2}")
      socket.close
    end
  end
end

$first_read = Proc.new do |socket|
  input = socket.gets
  socket.puts("1": #{input}")
  $work[socket] = $second_read
end

$seond_read = Proc.new do |socket|
  input = socket.gets
  socket.puts("2: #{input}")
  socket.close
  $readers.delete(socket)
  $work.delete(socket)
end

$work[$server] = $accept_socket

while true
  ready_readers = IO.select($readers)[0]
  ready_readers.each do |rr|
    $work[rr].call(rr)
  end
end
```
* async_read can be nested as deeply as we want. Just remember to close the socket. A lot of indentations like that is called "callback hell."
* Node follows this kind of pattern, where you always pass callbacks to functions. Everything is asynchronous, and there are hardly any synchronous methods.
* Asynchronous calculator :-) :
```ruby
require 'socket'
$server = TCPServer.new(3000)
$readers = [$server]
$work = {}

def async_read(socket, &prc)
  $readers << socket
  $work[socket] = Proc.new do
    $readers.delete(socket)
    $work.delete(socket)
    prc.call
  end
end

$accept_socket = Proc.new do |server|
  socket = server.accept
  async_read(socket) do
    input1 = socket.gets
    num1 = Integer(input1)
    async_read(socket) do
      input2 = socket.gets
      num2 = Integer(input2)
      socket.puts(num1 + num2)
      socket.close
    end
  end
end

$first_read = Proc.new do |socket|
  input = socket.gets
  socket.puts("1": #{input}")
  $work[socket] = $second_read
end

$seond_read = Proc.new do |socket|
  input = socket.gets
  socket.puts("2: #{input}")
  socket.close
  $readers.delete(socket)
  $work.delete(socket)
end

# THIS IS CALLED AN EVENT REACTOR:
$work[$server] = $accept_socket

while true
  ready_readers = IO.select($readers)[0]
  ready_readers.each do |rr|
    $work[rr].call(rr)
  end
end
```

## 05 Node Video

* calc-app.js
```javascript
var net = require('net');
var server = net.createServer();
server.on('connection', function (socket) {
  socket.on('data', function (data1) {
    var i1 = parseInt(data1);
    socket.once("data", function(data2) {
      var i2 = parseInt(data2);
      socket.write((i1+i2).toString()), function() {
        socket.end();
      });
    });
  });
});
server.listen(3000);
```
* Node has an event reactor written and working behind the scenes.
* This works and is non-blocking.
* Node doesn't have a way to create new threads. Node forces you to use the non-blocking style pattern (callbacks).

### Node Trivia

* Node was written to solve the C10K problem (how to handle 10k clients simultaneously)
* Node is designed to support this by not creating any new threads; just more events (sockets) to listen to!
* Node is a good choice for building applications that handle many simultaneous connections.
