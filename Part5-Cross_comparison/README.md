# Part 5. Cross Comparison
終於進入我們的最終章，一個大亂鬥的時代😂

進到 Part 5 資料夾，執行
```bash
make
```
就可以自動編譯所有的程式及輸出比較表格與圖片
首先先來看輸出的表格以及圖片

![](https://i.imgur.com/jItYoZk.png)

|Rank  |         program |    real  |  user  |system  |Execution Time|
|----| ---- | ----| ----  | ----  |----|
|0   |     interpreter | 2:18.71  |129.01  |  0.14  |        129.15|
|1   |interpreter_opt1 | 1:02.82  | 61.48  |  0.01  |         61.49|
|2   |interpreter_opt3 | 0:53.23  | 50.54  |  0.12  |         50.66|
|3   |interpreter_opt2 | 0:47.69  | 46.79  |  0.03  |         46.82|
|4   |             sed | 0:19.47  | 18.17  |  0.03  |         18.20|
|5   |      jit_opcode | 0:04.35  |  3.89  |  0.03  |          3.92|
|6   |        compiler | 0:04.20  |  3.85  |  0.00  |          3.85|
|7   |      jit_dynasm | 0:04.75  |  3.81  |  0.03  |          3.84|
|8   | jit_dynasm_opt2 | 0:01.37  |  1.25  |  0.01  |          1.26|
|9   | jit_dynasm_opt1 | 0:01.46  |  1.25  |  0.00  |          1.25|
|10  |          sed_O2 | 0:01.21  |  1.20  |  0.00  |          1.20|
|11  |          sed_O1 | 0:01.20  |  1.17  |  0.01  |          1.18|
|12  |          sed_O3 | 0:01.18  |  1.17  |  0.00  |          1.17|

可以發現我們從頭到尾的心路歷程，從最簡單的 interpreter 一路走到 JIT compiler，最後我們經過優化的JIT compiler 和 編譯器方法優化的速度幾乎一致，及時編譯器的效果很猛吧😁，接下來三個小節就來探討各個不同比較


## 5.1 Interpreter 家族

![](https://i.imgur.com/nlEJB7f.png)
|Rank  |         program |    real  |  user  |system  |Execution Time|
|----| ---- | ----| ----  | ----  |----|
|0   |     interpreter | 2:18.71  |129.01  |  0.14  |        129.15|
|1   |interpreter_opt1 | 1:02.82  | 61.48  |  0.01  |         61.49|
|2   |interpreter_opt3 | 0:53.23  | 50.54  |  0.12  |         50.66|
|3   |interpreter_opt2 | 0:47.69  | 46.79  |  0.03  |         46.82|
|4   |             sed | 0:19.47  | 18.17  |  0.03  |         18.20|

可以發現，interpreter 先經由 jumptable 的優化後，效率直接提升一倍，之後再加上`contineous_count` 的運算壓縮後，又快了大約 15 秒，而再加入處理 loop pattern 後，效率反而慢了一點點。但是這些都還沒有 sed 先轉成
 C code 再編譯的方法快，原因是因為 interpreter 儘管最佳化，仍需要讀取 BF code 外，每個運算都要經過至少兩個 branch 的指令(for loop + switch case)，而編譯後的 sed 少了很多 branch 指令，自然而然指令數量少，執行效率也會加快很多。這就是為什麼 interpreter 仍然效率輸 sed 的方法的理由。

## 5.2 最佳化 Interpreter vs  Simple Dynasm JIT
![](https://i.imgur.com/bgnQT0Z.png)
|Rank  |         program |    real  |  user  |system  |Execution Time|
|----| ---- | ----| ----  | ----  |----|
|0   |interpreter_opt1 | 1:02.82  | 61.48  |  0.01  |         61.49|
|1   |interpreter_opt3 | 0:53.23  | 50.54  |  0.12  |         50.66|
|2   |interpreter_opt2 | 0:47.69  | 46.79  |  0.03  |         46.82|
|3   |      jit_dynasm | 0:04.75  |  3.81  |  0.03  |          3.84|

將 interpreter 引入了 JIT 的技術，在執行時間產生機器碼直接執行，可以看到，無論我們的直譯器做了多少最佳化，真的就是被 JIT 屌打😂。效率直接高了近 15 倍。所以 JIT compiler 的確可以幫我們大幅加速直譯器的執行效率😀。


## 5.3 站在顛峰 - 最快的辣些人
![](https://i.imgur.com/7WItIlM.png)
|Rank  |         program |    real  |  user  |system  |Execution Time|
|----| ---- | ----| ----  | ----  |----|
|0   |      jit_opcode | 0:04.35  |  3.89  |  0.03  |          3.92|
|1   |        compiler | 0:04.20  |  3.85  |  0.00  |          3.85|
|2   |      jit_dynasm | 0:04.75  |  3.81  |  0.03  |          3.84|
|3   | jit_dynasm_opt2 | 0:01.37  |  1.25  |  0.01  |          1.26|
|4   | jit_dynasm_opt1 | 0:01.46  |  1.25  |  0.00  |          1.25|
|5   |          sed_O2 | 0:01.21  |  1.20  |  0.00  |          1.20|
|6   |          sed_O1 | 0:01.20  |  1.17  |  0.01  |          1.18|
|7   |          sed_O3 | 0:01.18  |  1.17  |  0.00  |          1.17|

在還沒做任何最佳化前(contineous_count, loop pattern)，我們實做的方法最多只能到4秒左右，但是加入了 Part4 提到的最佳化技術後，整體又快了3秒。執行效率1秒多。使得我們實做的 just in time compiler 逼近 clang 編譯器進行程式碼最佳化後的效率。很厲害吧!😉


## 5.4 結尾和未來工作
我們實作了不同的方法去執行運算量大的碎形 brainfuck 程式檔，最後利用及時編譯技術加上一些最佳化方法，把普通的 interpreter 效率提升 120 倍，執行只需要 1秒，可以說是非常值得使用的技術。至於未來工作是加入 LLVM 及 asmjit 這兩個工具幫我們建立 JIT 編譯器，並且加入比較，且把組合語言的基礎打好😀。

## 5.5 參考連結 (特別感謝的網站)
1. [Interpreter, Compiler, JIT](https://nickdesaulniers.github.io/blog/2015/05/25/interpreter-compiler-jit/)

2. [JIT 編譯器原理簡述/實現Brainfuck JIT 編譯器](http://accu.cc/content/jit_tour/brainfuck_jit/)

3. [2016q3 Homework5 - JIT compiler](https://hackmd.io/@nKngvyhpQdGagg1V6GKLwA/HJjoxbvke?type=view#2016q3-Homework5---JIT-compiler)
4. [虛擬機器設計與實作](https://hackmd.io/@sysprog/SkBsZoReb?type=view)
5. [Adventures in JIT compilation: Part 1 - an interpreter](https://eli.thegreenplace.net/2017/adventures-in-jit-compilation-part-1-an-interpreter/)

6. [Adventures in JIT compilation: Part 2 - an x64 JIT](https://eli.thegreenplace.net/2017/adventures-in-jit-compilation-part-2-an-x64-jit/)