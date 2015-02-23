#SpwanedProcess
主要是參考erlang的風格，提供另外一種透過notify和receive的defer運作方式，要注意的是被notify後並不會立即執行，EventMachine 會自動幫你排入程序等待被運作
```Ruby
EM.run {
  pong = EM.spawn {|x, ping|
    puts "Pong received #{x}"
    ping.notify( x-1 )
  }

  ping = EM.spawn {|x|
    if x > 0
      puts "Pinging #{x}"
      pong.notify x, self
    else
      EM.stop
    end
  }

  ping.notify 3
}
```
從上面的範例可以看到ping.notify 觸發ping 以後，透過ping pong的notify method 互相丟訊息，達到互動的效果。
