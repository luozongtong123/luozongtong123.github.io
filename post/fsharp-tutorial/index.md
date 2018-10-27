<!--more-->

``` fsharp
let doFuns() =
  let getSum(x : int, y : int) :int = x + y

  printfn "5 + 7 = %i" (getSum(5,7))

  // 递归函数
  let rec factorial x =
    if x < 1 then 1
    else x * factorial (x - 1)
  
  printfn "Factorial 4 : %i" (factorial 4)
  // 1st : result = 4 * factoral(3) = 4 * 6 = 24
  // 2st : result = 3 * factoral(2) = 3 * 2 = 6
  // 1st : result = 2 * factoral(1) = 2 * 1 = 2

  // 定义一个列表 (List)
  let randList = [1; 2; 3]
  // map 可以对数组里的每一个元素进行特定操作，
  // 第一个参数是 lambda 表达式 (anonymous function, 匿名函数)
  // 第二个参数是被操作的数组
  let randList2 =  List.map (fun x -> x * 2) randList
  printfn "Double List : %A" randList2

  // 管道可以很方便地进行多步计算
  // 管道：将值输入(|>)到函数中
  [5; 6; 7; 8]
  |> List.filter (fun v -> (v % 2) = 0)
  |> List.map (fun x -> x * 2)
  |> printfn "Even Doubles %A" 

  // >> 和 << 可以将两个函数合二为一
  let multNum x = x * 3
  let addNum y = y + 5

  let multAdd = multNum >> addNum
  let addMult = multNum << addNum

  printfn "multAdd : %i" (multAdd 10)
  printfn "addMult : %i" (addMult 10)

doFuns()
```

