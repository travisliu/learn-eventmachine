#Run
我們前一直重複提到在Reactor pattern 扮演重要角色的 event loop，就是在 EM.run 裡註冊要處理的事件與處理方式，在每次事件觸發時，內部的Service handler 會分配到各個Event handler

```Ruby
puts "start"
EM.run do {
  puts "Init"
  EM.add_timer(1) do
    puts "Quit"
    EM.stop_event_loop
  end
  puts "OK"
end
puts "done"

> start
> init
> OK
> Quit
> done
```
從上面的執行結果我們可以看到，最一開始的puts "start"後隨即進入run的block，也就開始了的event loop，跟著顯示了"init"，而下面的EM.add_timer 則是用來指定多少秒數後要執行裡面的code，所以跟著顯示的是OK，到這裡就停住了，因為現在已經在event loop裏也就不會跟著顯示"done"，剛剛指定的一秒過後進入了add_timer的block 顯示了 "Quit"，隨即使用EM.stop_event_loop 停止下來，注意 EM.stop_event_loop 其實跟 EM.stop 一樣的東西，都是用來停止Event loop 用的，隨後你就可以看到"done"被顯示出來，整個程式就完成了。
