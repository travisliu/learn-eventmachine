# 什麼是reactor pattern
Reactor pattern 的目的是在於解決IO Blocking的問題

用容易理解的比喻, 一般常用線性的開發方式，就像一個服務生接受了一位客人的點餐後，將菜單交給了廚師，他會站在廚房門口等待廚師烹調完客人的餐點，再將餐點送至客人的餐桌上，等這一位客人服務完後，才會接著接受下一位客人的點餐，而這位服務生在將菜單送至廚師的手上等待烹飪的過程中，空閒了下來不做其他的事情，就是reactor pattern 要解決的IO Blocking

在線性程式裡我們常遇到這樣的狀況，web application 的Process 接到了request ，處理過後接著要讀取資料或是寫入資料到資料庫，中間可能等待Database處理的時間（可能十幾毫秒或更長）該Process其實是處於sleeping 的狀態不做事情的，這也是為什麼你可能會發現server的request處理量一直上不去，但CPU卻明明沒有吃滿，這就因為Process 在處理資料庫的存取、檔案的IO或者是跟其他Service的溝通之間的等待時間，其實該Process都處於Sleeping 狀態。

聰明的你可能會想到，為什麼服務生在等待廚師烹調的過程中，不去服務其他的顧客呢？對，這就是Reactor pattern 解決IO Blocking 的方式，去提升Process的處理效能，在服務生將菜單交給了廚師以後時，他可以接著服務其他顧客，只要留神廚師是否已經把料理烹飪好後呼喚他將菜送至客人的餐桌上？這樣的行為在Reactor Pattern 稱之為Defer。

在Reactor Pattern 中有一個Single thread 的event loop，不停的循環等待是否有新的事件進來要處理，事件處理的過程中如有IO Blocking的產生，像是要讀取資料庫裡的資料，他會使用defer 將這個動作丟到event loop以外的另一個thread，讓他在等待資料庫讀取的sleeping是發生在另外一個thread，而不會讓event loop 產生阻塞並且繼續服務下一個事件，等待資料庫讀取完了資料回傳回來以後，再透過callback 將資料傳回event loop 這個thread 中繼續處理。

注意這裏有很重要的觀念就是整個所有的邏輯都是在single thread 中完成，是只有在有可能因為IO Blocking 產生sleeping的狀況才把它拋至另外一個thread，好讓event loop可以繼續處理其他的event，如果是大量的資料需要使用較長時間的CPU time，則應該使用eventmachine 裡的next_tick而非是defer到另外一個thred處理。

或許看到這你會有個問題，IO Blocking透過 multi thread的方式處理一樣能將CPU吃滿解決sleeping的狀況，為什麼要這麼麻煩？ Reactor pattern 的single thread跟threads的處理方式的不同的地方，在於threads是屬於搶占式調度( preemptive scheduling )，在考慮到Race conditions的狀況下，可以選則使用Reactor pattern，能在single-threaded process 中掌控一系列的事件的處理順序。

Race conditions
http://zh.wikipedia.org/wiki/%E7%AB%B6%E7%88%AD%E5%8D%B1%E5%AE%B3
