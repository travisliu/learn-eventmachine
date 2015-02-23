#Deferrable
Deferrable 不同於defer，不要搞混了，它主要的目的是在於管理callback，共分成 succeeded 與 failed 兩種結果，會分別回傳至 callback 和 errback 兩種 method，而一個 Deferrable 物件可以帶多個的 callback 或 errback，可以參考以下範例

```Ruby
class Ping
  include EM::Deferrable

  def wrang_way(param)
  	# set_deferred_status(:failed,  param)
  	# set_deferred_failure param
  	fail param
  end

  def right_way(param)
  	# set_deferred_status(:succeeded,  param)
  	# set_deferred_success param
    succeed param
  end
end

EM.run do
  png = Ping.new

  png.callback do |res|
  	puts "in callback with behavior:" + res
  end

  png.callback do |res|
  	puts "in callback2: with behavior:" + res
  	png.wrang_way 'stealing'
  end

  png.errback do |res|
  	puts "in errback with behavior:" + res
  end

  png.errback do |res|
  	puts "in errback2 with behavior:" + res
  	EM.stop
  end

  png.right_way 'contribution'
end
```
從範例中可以看到callback 和 errback 各帶了兩，兩個在條件符合的狀況下都會執行，right_way是透過set_deferred_status 帶succeeded 的symbol 就會回傳至callback method，另外也提供了像set_deferred_success 和succeed兩個較簡短的sugar可以使用，failed則是反之亦然。

###em-http-request
看到這裡已經學會足夠的工具用EventMachine 來做最基本的應運，這裡要介紹的[em-http-request](https://github.com/igrigorik/em-http-request) 就是在EventMachine 底下透過 Deferrable 實現的HTTP client，利用這它我們就能簡單的做出避開等待http response的blocking

```Ruby
urls = ARGV
if urls.size < 1
  puts "Usage: #{$0} <url> <url> <...>"
  exit
end

pending = urls.size

EM.run do
  urls.each do |url|
    http = EM::HttpRequest.new(url).get
    http.callback {
      puts "#{url}\n#{http.response_header.status} - #{http.response.length} bytes\n"
      puts http.response

      pending -= 1
      EM.stop if pending < 1
    }
    http.errback {
      puts "#{url}\n" + http.error

      pending -= 1
      EM.stop if pending < 1
    }
  end
end
```
帶入你所有想要取得內容的url後，em-http-request 就會依序幫你取得網頁內容，運作的方式就是在送出第一筆request後，在等待response的時間就會繼續向下一筆url 送出request，達到效能最佳化的目的，等收到response後，會依執行結果帶入callback 或是errback。
