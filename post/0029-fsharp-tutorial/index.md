  

This is a F# Tutorial.
<!--more-->

``` fsharp
open System                        // 这步操作可以让我们使用 .NET Core 的函数库
open System.Drawing

let hello() =                      // let 用来绑定函数名或者参数名
  printf "Enter your name: "     // printf 不启用新行的打印函数
    
  let name = Console.ReadLine()  // 读取一行并将读取到的值和 name 绑定

  printfn "Hello %s!" name       // 启用新行的打印 % 引用参数
  // %s %i %f %b %A %O           

hello()

let hello1() =
  printfn "PI: %f" 3.14159265358979323846264338327       // 浮点数只有6位小数
  printfn "PI: %.4f" 3.14159265358979323846264338327     // 可以调整显示的小数的位数
  printfn "Big PI: %M" 3.14159265358979323846264338327M  // 最高可以到达27位小数

hello1()

let hello2() =
  printfn "%-5s %5s" "a" "b"             // 添加空格
  printfn "%*s" 10 "Hi"                  // 动态空格
    
hello2()

let bindStuff() =
  let mutable weight = 175         // 可以改变的参数 (mutable)
  weight <- 170                    // 使用 <- 赋值
  
  printfn "Weight: %i" weight

  let changeMe = ref 10            // 奇怪的语法，先不管这了
  changeMe := 50

  printfn "Change: %i" ! changeMe

bindStuff()

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

// 一些数学函数
let doMath() =
  printfn "5 + 4 = %i" (5 + 4)
  printfn "5 - 4 = %i" (5 - 4)
  printfn "5 * 4 = %i" (5 * 4)
  printfn "5 / 4 = %i" (5 / 4)
  // "" 里面用 % 要用 %%
  printfn "5 %% 4 = %i" (5 % 4)
  // 5^2
  printfn "5 ** 2 = %f" (5.0 ** 2.0)

  let number = 2
  printfn "Type: %A" (number.GetType())
  printfn "A Float: %.2f" (float number)
  printfn "An Int: %i" (int 3.14)

  // 其他数学函数
  // 绝对值
  printfn "abs 4.5 : %i" (abs -1)
  // 向上取整
  printfn "ceil 4.5 : %f" (ceil 4.5)
  // 向下取整
  printfn "floor 4.5 : %f" (floor 4.5)
  // ln
  printfn "log 2.71828 : %f" (log 2.71828)
  // lg
  printfn "log10 1000 : %F" (log10 1000.0)
  // 求平方根
  printfn "sqrt 25 : %f" (sqrt 25.0)


doMath()

let stringStuff() =
  let str1 = "This is a random string\\"
  // @ 加到字符串之前可以将 \ 加入到字符串之中
  let str2 = @"I ignore backslashes\\\\"
  // 三个双引号可将引号添加到字符串
  let str3 = """ "I ignore qutes and backslashes" """
  // 字符串相连
  let str4 = str1 + " " + str2
  // length 获取字符串的长度
  printfn "Length : %i" (String.length str4)
  // 和 C 相似的字符访问
  printfn "%c" str1.[1]
  // 和 Python 相似的字符串切片操作
  printfn "First word : %s" (str1.[..3])
  printfn "Second word : %s" (str1.[5..6])
  printfn "The rest word : %s" (str1.[8..])

  // 
  let upperStr = String.collect (fun c -> sprintf "%c, " c) "commas"
  printfn "Commas : %s" upperStr

  // 测试字符串中是否有字符满足条件
  printfn "Any upper: %b" (String.exists (fun c -> Char.IsUpper c) str1)

  // 测试字符串中所有的字符满足条件
  printfn "Number : %b" (String.forall (fun c -> Char.IsDigit c) "1234")

  // 初始化字符串
  let string1 = String.init 10 (fun i -> i.ToString() )
  printfn "Numbers : %s" string1

  // 对字符串中的每个字符进行操作
  String.iter (fun c -> printfn "%c" c) "Print Me!"

stringStuff()

let loopStuffWhile() =
  let magic_num = "7"
  let mutable guess = ""

  while not (magic_num.Equals(guess)) do
    printf "Guess the Number :"
    guess <- Console.ReadLine()

  printfn "You Guessed the Number!"

loopStuffWhile()

let loopStuffFor() = 
  for i = 1 to 10 do 
    printfn "%i" i

  for i = 10 downto 1 do
    printfn "%i" i

  for i in [1..10] do
    printfn "%i" i

  [1..10] |> List.iter (printfn "Num : %i")

  let sum = List.reduce (+) [1..10]
  printfn "Sum : %i" sum

loopStuffFor()

let cond_stuff() =
  let age = 8

  if age < 5 then
    printfn "Preschool"
  elif age = 5 then
    printfn "Kindergarten"
  elif (age > 5) && (age <= 18) then
    let grade = age - 5
    printfn "Go to grade %i" grade
  else
    printfn "Go to College"

  let gpa = 3.9
  let income = 15000

  printfn "College Grant: %b" ((gpa >= 3.8) || (income <= 12000))

  printfn "Not True: %b" (not true)

  let grade2: string =
    match age with
    | age when age < 5 -> "Preschool"
    | 5 -> "Kindergaten"
    | age when ((age > 5) && (age <= 18)) -> (age - 5).ToString()
    | _ -> "Collage"

  printfn "Grade2 : %s" grade2

cond_stuff()

let list_stuff() =

  let list1 = [1; 2; 3; 4]
  list1 |> List.iter (printfn "Num : %i")
  printfn "list1 : %A" list1

  let list2 = 5::6::7::[]
  printfn "list2 : %A" list2 

  let list3 = [1..5]
  printfn "list : %A" list3

  let list4 = ['a'..'g']
  printfn "list4 : %A" list4

  let list5 = List.init 5 (fun i -> i * 2)
  printfn "list5 : %A" list5

  let list6 = [for a in 1..5 do yield (a * a)]
  printfn "list6 : %A" list6

  let list7 = [for a in 1..20 do if a % 2 = 0 then yield a]
  printfn "list7 : %A" list7

  let list8 = [for a in 1..3 do yield! [a..a + 2]]
  printfn "list8 : %A" list8

  printfn "Length : %i" list8.Length
  printfn "Empty : %b" list8.IsEmpty
  printfn "Index 2 : %c" (list4.Item(2))
  printfn "Head : %c" (list4.Head)
  printfn "Tail : %A" (list4.Tail)

  let list9 = list3 |> List.filter (fun x -> x % 2 = 0)

  let list10 = list9 |> List.map (fun x -> (x * x))

  printfn "Sorted : %A" (List.sort [5; 4; 3])

  printfn "Sum : %i" (List.fold (fun sum elem -> sum + elem) 0 [1; 2; 3])

list_stuff()

type emotion =
| joy = 0
| fear = 1
| anger = 3

let enum_stuff() =
  let myFeeling = emotion.joy

  match myFeeling with
  | joy -> printfn "I'm joyful"
  | fear -> printfn "I'm fearful"
  | anger -> printfn "I'm angry"

enum_stuff()

let optionStuff() =
  let divide x y =
    match y with
    | 0 -> None
    | _ -> Some(x / y)

  if (divide 5 0).IsSome then
    printfn "5/0 = %A" ((divide 5 0).Value)
  elif (divide 5 0).IsNone then
    printfn "Can't Divide by Zero"
  else
    printfn "Something happenend"

optionStuff()

let tupleStuff() =
  let avg (w, x, y, z) : float =
    let sum = w + x + y + z
    sum / 4.0

  printfn "Avg : %f" (avg (1.0, 2.0, 3.0, 4.0))

  let my_data = ("luo", 22, 6.25)

  let (name, age, _) = my_data

  printfn "Name : %s, age : %i" name age

tupleStuff()

type customer =
  {Name : string;
   Blance : float}

let recordStuff() = 
  let bob = {Name = "Bob Smith"; Blance = 101.5012}
  printfn "%s owes us %.2f" bob.Name bob.Blance

recordStuff()

let seqStuff() = 
  let seq1 = seq {1..100}
  let seq2 = seq {0..2..50}
  let seq3 = seq {50..1}

  printfn "%A" seq2

  Seq.toList seq2 |> List.iter (printfn "Num : %i")

  let isPrime n =
    let rec check i =
      i > (n / 2) || (n % i <> 0 && check (i + 1))
    check 2

  let prime_seq = seq {for n in 1..500 do if isPrime n then yield n}

  printfn "%A" prime_seq

  Seq.toList prime_seq |> List.iter (printfn "Prime : %i")
  
seqStuff()

let mapStuff() =
  let customers =
    Map.empty.
      Add("Bob Smith", 100.50).
      Add("Sally Marks", 50.25)

  printfn "# of Customers %i" customers.Count

  let cust = customers.TryFind "Bob Smith"
  match cust with
  | Some x -> printfn "Balace : %.2f" x
  | None -> printfn "Not Found"

  printfn "Customer : %A" customers

  if customers.ContainsKey "Bob Smith" then
    printfn "Bob Smith was found"

  printfn "Bobs Blance : %.2f" customers.["Bob Smith"]

  let custs2 = Map.remove "Sally Marks" customers

  printfn "# of Customers %i" custs2.Count

mapStuff()

let addStuff<'T> x y =
  printfn "%A" (x + y)

let genericStuff() =
  addStuff<float> 5.5 2.4
  // addStuff<int> 4 2

genericStuff()

let expStuff() =
  let divideFloat x y =
    try 
      printfn "%.2f / %.2f = %.2f" x y (x / y)
    with 
      | :? System.DivideByZeroException as ex ->
        printfn "Can't Divide by Zero"
  divideFloat 5.0 0.0

  let divideInt x y =
    try 
      printfn "%i / %i = %i" x y (x / y)
    with 
      | :? System.DivideByZeroException as ex ->
        printfn "Can't Divide by Zero"
  divideInt 5 0

  let divideIntR x y =
    try 
      if y = 0 then raise(DivideByZeroException "Can't Divide by 0")
      else
        printfn "%i / %i = %i" x y (x / y)
    with 
      | :? System.DivideByZeroException as ex ->
        printfn "Can't Divide by Zero"
  divideIntR 5 0

expStuff()

type Rectangle = struct
  val Length : float
  val Width : float
  new (length, width) =
    {Length = length; Width = width;}
end

let structStuff() =
  let area(shape: Rectangle) =
    shape.Length * shape.Width

  let rect = new Rectangle(5.0, 6.0)

  let rect_area = area rect

  printfn "Area : %.2f" rect_area

structStuff()

type Animal = class
  val Name : string
  val Height : float
  val Weight : float

  new (name, height, weight) =
    {Name = name; Height = height; Weight = weight;}

  member x.Run =
    printfn "%s Runs" x.Name
end

type Dog(name, height, weight) =
  inherit Animal(name, height, weight)

  member x.Bark = 
    printfn "%s Barks" x.Name

let classStuff() =
  let spot = new Animal("Spot", 20.5, 40.5)
  spot.Run

  let bowser = new Dog("Bowser", 20.5, 40.5)
  bowser.Run
  bowser.Bark

classStuff()

Console.ReadKey() |> ignore        // 输入一个字符，防止命令行一闪而过

```

Gist: 

<script src="https://gist.github.com/zt-luo/d6cc2dc11eb5a1548ed84b5218215636.js"></script>

