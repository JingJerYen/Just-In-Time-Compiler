# Just In Time Compiler

# Why Compiler ?  
1. 為什麼需要編譯器 ? 為了更快、更有效率的使用硬體，如 CPU、GPU等，也希望程式碼可以跑在不同硬體平台上 

2. 哪裡有編譯器的影子 ? 到處都是。如 Google V8, Pypy, Flacc, Java, .NET, Android ART, OpenGL ([where to find compiler](http://slide.logan.tw/compiler-intro/#/4/2))


3. LLVM 編譯器框架，因為統一了 IR 以及對 IR 的優化，再也不用因為執行平台的不同要花大量時間重新設計，可以幫助我們快速建立一個 Compiler，現在很多編譯器都使用 LLVM 開發，LLVM 也有豐富的編譯輸出 (ARM, x86, Alpha, PowerPC)，LLVM的出現，讓不同的前端后端使用統一的LLVM IR ,如果需要支持新的程式語言或者新的設備平台，只需要開發對應的前端和後端即可。同時基於LLVM IR我們可以很快的開發自己的程式語言。

   ![](https://pic3.zhimg.com/80/v2-64db6352bd23eb839ea4517ff70f2ba2_1440w.png)

    也因此，**LLVM統一的IR是它成功的關鍵之一，也充分說明了一個優秀IR的重要性**。IR可以說是一種膠水語言，注重邏輯而去掉了平台相關的各種特性，這樣為了支持一種新語言或新硬體都會非常方便

4. 根據[此篇文章](https://zhuanlan.zhihu.com/p/65452090)提到有限的精力跟無限的算力，深度學習編譯技術才會越來越重要，人會特定去最佳化某些算子(卷積)，而不一定適用網路的每一層。但深度學習編譯器可以幫助我們進行針對網路的每一層進行最佳化，通過（接近無限）的算力去適配每一個應用場景看到的網絡，這是深度學習編譯器比人工路線強的地方。**編譯器可以達到更多的自動化**，以下節錄至該文章
   >當比較TVM和傳統方法的時候的時候，我們往往會發現：在標準的benchmark（如imagenet resnet）上，編譯帶來的提升可能只在10%到20%，但是一旦模型相對不標準化（如最近的OctConv，Deformable,甚至是同樣的resnet不同的輸入尺寸），編譯技術可以帶來的提升會非常巨大。原因也非常簡單，有限的精力使得參與優化的人往往關注有限的公開標準benchmark，但是我們的部署目標往往並非這些benchmark，自動化可以無差別地對我們關心的場景進行特殊優化。接近無限的算力和有限的精力的差別正是為什麼編譯技術一定會越來越重要的原因。

   另外，一個較新的技術 - TVM，引入 `graph compiler`，將深度學習的模型轉成 graph IR，對 IR 進行最佳化後，可以跑在如手機這種硬體資源較低的移動式裝置(當然會捨棄精度減少計算量，在精度和效率之間進行取捨)。和傳統編譯器不同的是，這類編譯器不光要解決跨平台，還有解決對神經網絡本身的最佳化問題，這樣原先一層的IR就顯得遠遠不夠，原因在於如果設計一個方便硬件最佳化的低階的語言，你幾乎很難從中推理一些神經網路中高階的概念進行最佳化

![](https://pic3.zhimg.com/80/v2-cf84dfa43008de15457e188adca9a582_1440w.png) 

![](https://pic2.zhimg.com/80/v2-4f45f71b8e9e0338924e689a8c3b021d_1440w.png)
   另外TVM的對手，XLA（加速線性代數）是針對線性代數的特定領域編譯器，可優化TensorFlow計算。結果是在服務器和移動平台上提高了速度，記憶體使用率和可移植性。

![](https://pic2.zhimg.com/80/v2-18d0443d567986dc4f34d23e4daa890d_1440w.jpg?source=1940ef5c)

   


總結來說，因為根據摩爾定律，硬體晶片效能約每隔兩年便會增加一倍，但是近來計算能力的需求爆炸性的增加，例如深度學習，以及計算機架構的多樣性，摩爾定律逐漸跟不上計算能力的需求，所以我們需要編譯器，針對有限資源的硬體及架構產出最佳化、效率最高的程式碼

# Compiler Language vs Interpreter Language
> [參考連結](https://github.com/yaofly2012/note/issues/193)

跟據翻譯的方式，分為編譯器和直譯器，以下為兩者比較

|     | 編譯器 | 直譯器 |
|-----|--------| --------|
|執行| 要先編譯再執行| 不用事先編譯 |
|啟動| 慢 (要等編譯) | 快(不用事先編譯) |
|執行效率| 快，編譯時還可以事先優化 | 慢 |
| 重編譯 | 不用 | 每次啟動都要重編譯 |
| 跨平台 | 差，需要重編譯  | 好

另外像 JVM，在 編譯 和 直譯 之間做一個取捨
1.  先編譯成中間文件 ( Bytecode)
2.  再直譯執行，並搭配 JIT compiler 的 runtime 優化

此時 JIT Compiler 引入，使我們可以達到動態編譯的技術
![](https://camo.githubusercontent.com/a19600604970dcbe59dec105638e9607e3e3ffccf5fd4512f74c13facb77ca2e/68747470733a2f2f696d6167652d7374617469632e7365676d656e746661756c742e636f6d2f3235362f3339392f3235363339393336352d356638326235653666303438615f61727469636c6578)

 
`Baseline compiler` : 編譯的單位是程式碼行（stub）

`Optimizing compiler` : 編譯和優化的單位為函數



# 專案目標
|Part|目標| 進度 |
|----|----| ----- | 
|Part 1. Simple JIT| 利用 Dynasm 幫助建立簡單的 JIT | ✔
|Part 2. BF compiler| 建立簡單的 Brainfuck compiler 和 interpreter | 💨(ing)|
|Part 3. BF JIT| 利用 Dynasm 幫助建立 BF JIT，以及做一些優化測試 | ❌ |


# 參考連結
1. [Deep Learning的IR之爭](https://zhuanlan.zhihu.com/p/29254171)

2. [JS 及時編譯器 Just-In-Time (JIT) compilers](https://github.com/yaofly2012/note/issues/193)

3. [What Can Compilers Do for Us?](https://www.slideshare.net/jserv/what-can-compilers-do-for-us/16-LLVM)

4. [為什麼這麼多人喜歡寫編譯器？](https://www.zhihu.com/question/39304476)

5. [虛擬機器設計與實作](https://hackmd.io/@sysprog/SkBsZoReb?type=view)

6. [深度學習編譯技術的現狀和未來](https://zhuanlan.zhihu.com/p/65452090)

7. [深度學習編譯器學習筆記和實踐體會 (好專欄!) ](https://www.zhihu.com/column/c_1169609848697663488)