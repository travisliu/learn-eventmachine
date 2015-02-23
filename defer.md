#Defer
所有會讓event loop產生blocking 的部分我們都會丟到defer裡，他會將他拋出event loop以外的另外一個thread 去等待，以避免阻塞的產生
Defer主要是以兩個部分組成，其中callback 就是將等待到的資料帶回event loop的thread 中 繼續接受處理，EventMachine 裡會為Defer

```Ruby
puts Thread.current
EM.run do
  puts "in run:" + Thread.current.to_s
  operation = proc {
    puts "in defer operation:" + Thread.current.to_s
    "result"
  }

  callback = proc {|result|
    puts "in defer callback:" + Thread.current.to_s
  }

  EM.defer(operation, callback)
end

#> <Thread:0x0000010103b7c0>
#>in run:#<Thread:0x0000010103b7c0>
#>in defer operation:#<Thread:0x0000010288b348>
#>in defer callback:#<Thread:0x0000010103b7c0>
```
從上面的範例我們可以看到，在進入run block 跟在裡面時都是在 0x0000010103b7c0 的Thread 裏，在defer 被觸發後進入了就進入了operation 裡，注意到defer的operation裡是位在另外一個 0x0000010288b348 的Thread裡以避免主線程的阻塞，等待處理結過出來後就回傳至callback裡，回到了event log的thread 0x0000010103b7c0 繼續執行，從這段裡我們可以清楚得理解到 EventMachine 是如何實現Reactor Pattern。
