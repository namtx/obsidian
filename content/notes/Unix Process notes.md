---
title: "Unix Process notes"
---

### Process ID
Mỗi process đều có một unique ID, nó giống như số chứng minh nhân dân của mình vậy.
Thử chạy command sau ở `irb` (interactive ruby), ta có process đang chạy `irb` có ID là `96160`.

```ruby
puts Process.pid
 => 96160
```

Mở một terminal khác rồi chạy command sau, ta sẽ lấy được một số thông tin cơ bản về process đó thông qua ID.
```console
ps -p <process-id>
  PID TTY           TIME CMD
96160 ttys002    0:00.45 irb
```

### Parent process
Mỗi process chạy OS đều có process cha, nó có thể lấy process ID của process cha bằng `ppid`

```ruby
Process.ppid
 => 24569
```

Thử đoán xem process có ID `24569` là process nào?

```console
ps -p 24569
  PID TTY           TIME CMD
24569 ttys002    0:04.95 -zsh
```

Vì mình chạy `irb` thông qua `zsh` nên process cha của `irb` sẽ là `zsh`.

### File descriptor

> in the land of Unix ‘everything is a file’

Giống như mỗi process đều có một unique ID, bất cứ lúc nào chúng ta mở một resource (files, sockets, devices,...) trong một process, thì resource đó cũng được đánh số thứ tự `fileno`
`fileno` sẽ luôn là số nhỏ nhất mà chưa được sử dụng

Run command sau ở `irb`.

```ruby
passwd = File.open('/etc/passwd')
puts passwd.fileno
 => 3

hosts = File.open('/etc/hosts')
puts hosts.fileno
 => 4

# Khi close file, thì file descriptor cũng sẽ bị xoá
# fileno `3` sẽ có thể được sử dụng lại khi process mở một resource khác.
passwd.close

null = File.open('/dev/null')
puts null.fileno
 => 3
```

Chúng ta có thể thắc mắc ở đây
> `fileno` sẽ luôn là số nhỏ nhất mà chưa được sử dụng
 
ở trên chỉ thấy `fileno` là `3` là nhỏ nhất, vậy `0`, `1`, `2` đi đâu rồi?

Câu trả lời là, mỗi process khi chạy đều sẽ mở sẵn 3 resources. `STDIN`, `STDOUT`, `STDERR`

```ruby
puts STDIN.fileno
 => 0
puts STDOUT.fileno
 => 1
puts STDERR.fileno
 => 2
```
Đây là 3 resources tiêu chuẩn của mỗi process.
- `STDIN` (input) là để nhận input từ keyboard, pipes (nhận user input từ bàn phím)
- `STDOUT` (output) là để process có thể ghi output ra ngoài devices, (in kết quả ra terminal)
- `STDERR` (error) là để process có thể ghi lỗi ra bên ngoài (ghi lỗi ra file log chẳng hạn)

### Fork process

Để tạo một process con từ một process cha, UNIX cung cấp cho chúng ta một API `fork`

Process con sau khi fork từ process cha sẽ giống hệt process cha từ memory đến file descriptor.

Như đoạn đầu đã giới thiệu, process con sẽ có `ppid` là `pid` của process cha.

```ruby
# In ra process id hiện tại
puts "Parent process ID: #{Process.pid}"
# => Parent process ID: 99719

# fork một child process
fork do
	# In ra process id của child process
	puts Process.pid
	# => 99762

	# In ra parent process id của child process
	puts Process.ppid
	# => 99719
end
```

Process con sẽ thừa kế lại toàn bộ file descriptor đang mở của process cha. `fileno` cũng sẽ không hề thay đổi. Vì vậy nên process con có thể dùng chung file, sockets với process cha.

Process con cũng sẽ thừa kế toàn bộ memory của process cha.

Ví dụ, chúng ta có một app Rails trong RAM 500MB, thì khi fork một process, process con cũng sẽ chiếm thêm 500MB memory copy từ process cha.

Sử dụng `fork` cho phép chúng ta có thể tạo thêm nhiều App instances trong tích tắc.

Nhưng như đã nói ở trên, việc copy sẽ rất tốn kém về RAM.

UNIX optimize nhược điểm trên bằng cơ chế copy-on-write (CoW), sau khi `fork`, nếu process cha hoặc process con chưa modify lại data thì việc copy sẽ được delay, và chỉ được thực hiện khi process cha hoặc process con modify.

```ruby
arr = [1, 2, 3]

fork do
	# Bên trong process con
	# Tại thời điểm này, process con chưa modify `arr` nên nó sẽ tiếp tục đọc shared data từ process cha, không copy data
	puts arr
end
```

```ruby
arr = [1, 2, 3]
fork do
	# giống như ở phía trên, chưa có quá trình copy nào xảy ra cả
	arr << 4
	# Dòng code trên modify `arr` nên một bản copy của `arr` sẽ được tạo trong memory, và process con sẽ đọc từ đó, process cha vẫn tiếp tục đọc từ vùng nhớ của riêng nó, không liên quan đến process con.
end
```

### Orphaned Processes
Chuyện gì sẽ xảy ra nếu sau khi `fork` một process con, và trong khi process con đó đang chạy thì process cha bị kill? 

```ruby
# test_orphan.rb

fork do
	5.times do
		sleep 1
		puts "I'm an orphan"
	end
end

abort "Parent process dies..."
```

khi chạy file trên ở terminal bằng `ruby test_orphan.rb`, ta sẽ thấy parent process sẽ dừng ngay lập tức, nhưng mà process vẫn còn chạy tiếp, process con vẫn còn tiếp tục ghi output ra `STDOUT` (terminal)

Khi một parent process die, OS sẽ để nguyên process con, không làm gì nó cả, nó vẫn sẽ tiếp tục chạy.

Vậy, orphan child process thì có tác dụng gì?

- Daemon process: thông thường khi ta muốn chạy một long running processes như: database server hay web server, chúng ta sẽ chạy ở chế độ daemon. Daemon process đơn giản là process được chủ ý cho trở thành orphan process.
- Vậy làm sao chúng ta có thể giao tiếp với orphan process? ta có thể giao tiếp với orphan process bằng UNIX signals.

### Process.wait

Từ những ví dụ ở trên, ta có thể thấy process cha sau khi fork sẽ chạy song song với process con, và sẽ xảy ra những case như trên, khi process cha đã exit nhưng process con vẫn còn chạy.

Vậy làm sao process cha có thể quản lý được các process con?

Trong Ruby, chúng ta có thể sử dụng `Process.wait` ở proces cha để có thể chờ cho process con exit.

```ruby
fork do
	5.times do
		sleep 1
		puts "Child process"
	end
end

Process.wait
abort "Parent process exits..."
```

Output:
```console
Child process
Child process
Child process
Child process
Child process
Parent process exits...
```

`Process.wait` là blocking call, nghĩa là process cha sẽ dừng lại để chờ cho một process con exit, sau đó nó sẽ tiếp tục công việc của mình.

**Chờ nhiều process con???**

Do `Process.wait` chỉ chờ duy nhất 1 process con exit nên khi có nhiều process con, ta phải gọi `Process.wait` tương ứng với số process con đang chạy

```ruby
5.times do
	fork do
		sleep 1
		puts "Child process"
	end
end
5.times do
	Process.wait
end

puts "Parent process exits..."
```

#### Race condition

```ruby
# Fork ra 2 process con
2.times do
	fork do
		# Process con exit ngay lập tức
		abort "Children process finished"
	end
end

# Process cha chờ process con đầu tiên exit, sau đó sleep 5s
# Trong khi process cha đang sleep thì process con thứ 2 exit.
puts Process.wait
sleep 5

# Thông tin về process thứ 2 vẫn còn lúc sau đó
# OS đã lưu lại info của process ngay cả sau khi nó exit.
puts Process.wait
```

Có thể thấy, OS sẽ lưu lại info của process con và trả về cho process cha, ngay cả khi process con đã exit rất lâu.

### Zoombie process

Việc lưu lại info của process con và trả về cho process cha có một vấn đề, đó là mặc dù process con đã kết thúc từ rất lâu, nhưng nếu process cha không gọi `Process.wait` thì info về trạng thái của process con sẽ tồn tại mãi, khiến cho việc sử dụng tài nguyên của OS không hiệu quả.
Và process con sẽ trở thành `Zoombie process`.

Chúng ta có thể sử dụng `Process.detach(pid)` để có clean up.

`Process.detach` đơn giản chỉ tạo ra một thread và làm một việc đơn giản là chờ process con đó kết thúc.

[Process.detach](https://github.com/namtx/til/issues/148)