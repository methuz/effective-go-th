###Intro
บทความนี้เกิดจากการอ่านและบันทึกสิ่งที่ได้จากการอ่าน [Effective Go](https://golang.org/doc/effective_go.html) จึงไม่ได้ถอดความจากต้นฉบับทั้งหมด ถ้าใครพบข้อผิดพลาดหรือต้องการแก้ไขอะไรก็ตามให้ถูกต้อง สามารถ Fork แล้ว Pull Request เข้ามาได้เลยครับ
###Comment
การคอมเมนต์บนหัวของฟังก์ชันให้เริ่มต้นด้วยชื่อฟังก์ชันนั้นเสมอ เพื่อเวลาที่เราใช้คำสั่ง godoc หาข้อมูลแล้ว pipe ไปที่ grep จะได้หาเจอง่ายขึ้น ยกตัวอย่างเช่น

```
$ godoc regexp | grep parse
```
>Compile parses a regular expression and returns, if successful, a Regexp
    parsed. It simplifies safe initialization of global variables holding
    cannot be parsed. It simplifies safe initialization of global variables

เราจะเห็นคำว่า Compile ขึ้นต้นประโยค ทำให้จำได้ว่าสิ่งที่เราหาอยู่ในฟังก์ชัน Compile

### การใช้ If
ให้ใส่ `{` ตามหลังในบรรทัดเดียวกัน
```
if i < f() {
    g()
}
```
ใน Go Library จะไม่ใช้ else เพราะฟังก์ชันมักจะจบด้วย break, continue, goto, หรือ return
```
f, err := os.Open(name)
if err != nil {
    return err
}
codeUsing(f)
```
### การใช้ For
ใน Go นั้น `for` คือ `while` และไม่มี do while
```
// Like a C for
for init; condition; post { }

// Like a C while
for condition { }

// Like a C for(;;)
for { }
```

ตัวอย่าง Loop
```
sum := 0
for i := 0; i < 10; i++ {
    sum += i
}
```
หรือจะใช้ range ก็ได้ ถ้าจะ loop ใน array, slice, string หรือ map
```
for key, value := range oldMap {
    newMap[key] = value
}
```
ถ้าจะเอาแต่ key ก็ไม่ต้องใส่ตัวที่ 2 แต่ถ้าจะเอาแต่ตัวที่ 2 ให้ใส่ _ ที่ตัวแรก
```
sum := 0
for _, value := range array {
    sum += value
}
```
### Switch

Switch ใน Go ถ้าไม่ใส่อะไรจะเป็นการเช็ค True/False

```
func unhex(c byte) byte {
    switch {
    case '0' <= c && c <= '9':
        return c - '0'
    case 'a' <= c && c <= 'f':
        return c - 'a' + 10
    case 'A' <= c && c <= 'F':
        return c - 'A' + 10
    }
    return 0
}
```
แต่ถ้าใส่จะเช็ค match เหมือนปกติ และใช้ `,` คั่นได้
```
func shouldEscape(c byte) bool {
    switch c {
    case ' ', '?', '&', '=', '#', '+', '%':
        return true
    }
    return false
}
```
######* ปล. Case จะไม่มีการ fall through หรือตกไป case ด้านล่าง

บางทีเราต้อง break loop ใน if เราสามารถสั่ง `break` ตามด้วยชื่อ Lavel ได้
```
Loop:
    for n := 0; n < len(src); n += size {
        switch {
        case src[n] < sizeOne:
            if validateOnly {
                break
            }
            size = 1
            update(src[n])

        case src[n] < sizeTwo:
            if n+1 >= len(src) {
                err = errShortInput
                break Loop
            }
            if validateOnly {
                break
            }
            size = 2
            update(src[n] + src[n+1]<<shift)
        }
    }
```
สามารถเช็ค `type` ด้วย switch ได้ด้วย

```
var t interface{}
t = functionOfSomeType()
switch t := t.(type) {
default:
    fmt.Printf("unexpected type %T", t)       // %T prints whatever type t has
case bool:
    fmt.Printf("boolean %t\n", t)             // t has type bool
case int:
    fmt.Printf("integer %d\n", t)             // t has type int
case *bool:
    fmt.Printf("pointer to boolean %t\n", *t) // t has type *bool
case *int:
    fmt.Printf("pointer to integer %d\n", *t) // t has type *int
}
```
###Function

Function สามารถ return ได้มากกว่า 1 ค่า เช่น
```
func nextInt(b []byte, i int) (int, int) {
    for ; i < len(b) && !isDigit(b[i]); i++ {
    }
    x := 0
    for ; i < len(b) && isDigit(b[i]); i++ {
        x = x*10 + int(b[i]) - '0'
    }
    return x, i
}
```

เราสามารถตั้งชื่อของตัวแปรที่ `return` ได้ ซึ่งถ้าการ `return` นั้นไม่มี parameter ตัว function จะเอาค่าปัจจุบันของตัวแปรนั้นส่งไปให้เลย

```
func ReadFull(r Reader, buf []byte) (n int, err error) {
    for len(buf) > 0 && err == nil {
        var nr int
        nr, err = r.Read(buf)
        n += nr
        buf = buf[nr:]
    }
    return
}
```

###Defer

ฟังก์ชันที่ถูกเรียกโดย `defer` จะทำงานจริง ๆ ก่อนที่ฟังก์ชันที่ครอบอยู่นั้นจะทำการ `return` ซึ่งมีประโยชน์ในการเรียกฟังก์ชันที่จำเป็นต้องถูกเรียกเสมอไม่ว่าจะเกิดอะไรขึ้น อย่างเช่นการปิดไฟล์ ซึ่งมีประโยชน์คือเราจะไม่มีทางลืมปิดไฟล์และการเอาฟังก์ชันเปิดและปิดไว้ใกล้กันมันทำให้ดูง่ายกว่าการเอาไปไว้ท้ายฟังก์ชันหลัก

```
// Contents returns the file's contents as a string.
func Contents(filename string) (string, error) {
    f, err := os.Open(filename)
    if err != nil {
        return "", err
    }
    defer f.Close()  // f.Close will run when we're finished.

    var result []byte
    buf := make([]byte, 100)
    for {
        n, err := f.Read(buf[0:])
        result = append(result, buf[0:n]...) // append is discussed later.
        if err != nil {
            if err == io.EOF {
                break
            }
            return "", err  // f will be closed if we return here.
        }
    }
    return string(result), nil // f will be closed if we return here.
}
```

ตัวแปรที่ถูกส่งเข้าไปในฟังก์ชันที่ถูกเรียกโดย `defer` จะมีค่าเท่ากับที่ใส่ไปตอนแรก ไม่ใช่ค่าในตอนที่ฟังก์ชันนั้นถูกรันจริง ๆ และ `defer` จะทำงานแบบ Last In First Out

```
    for i := 0; i < 5; i++ {
        defer fmt.Printf("%d ", i)
    }
```

โค้ดนี้จะได้ผลลัพท์เป็น `4 3 2 1 0` ไม่ใช่ `4 4 4 4 4` หรือ `0 1 2 3 4`


###Data

######new

การใช้ new จะ Allocate Memory แต่ไม่ Initialize Memory (ให้ค่าเป็น 0) แล้ว return มาเป็น Address (Pointer) หรือ nil นั่นเอง

```
p := new(SyncedBuffer)  // type *SyncedBuffer
var v SyncedBuffer      // type  SyncedBuffer
```

บางครั้งค่า 0 อาจไม่เพียงพอ เราสามารถใช้ `composite literal` ในการทำ Constructor ได้ ข้างล่างคือ struct ของ type file

```
    type file struct {
    fd      int
    name    string
    dirinfo *dirInfo // nil unless directory being read
    nepipe  int32    // number of consecutive EPIPE in Write
    }
```
ถ้าอยากกำหนดค่าก็ให้ใช้แบบนี้ `File{fd, name, nil, 0}` เรียงตามลำดับ
แต่ถ้าไม่อยากเรียงก็ให้ใส่ label  เช่น `File{fd: fd, name: name}`

`composite literal` ยังสามารถใช้ในการสร้าง array, slice หรือ map ได้ด้วย
```
a := [...]string   {Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
s := []string      {Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
m := map[int]string{Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
```

######make
`make` เช่น make(T, args) นั้นใช้สร้าง slice map และ channels เท่านั้น และมันจะ `return` ค่าที่ initialized(ไม่ใช่ 0 เหมือน new) แล้วของประเภท T (ไม่ใช่ pointer เหมือน new)

```
make([]int, 10, 100)
```
ตัวอย่างข้างบนจะสร้าง array ขนาด 100 int แล้วสร้าง slice struct ขนาด 10 และมีขนาดสูงสุด 100 ชี้ไปที่ 10 ตัวแรก ของ array (งงไหม ผมก็งงเหมือนกัน ลองอ่านเกี่ยวกับ Slice ด้านล่างแล้วไปอ่านนี่ต่อเลย [Arrays, slices (and strings): The mechanics of 'append']()http://blog.golang.org/slices)
ลอง fmt.Println() ออกมาดู ได้ผลลัพท์เป็น `[0 0 0 0 0 0 0 0 0 0]`

 ตัวอย่างด้านล่างเป็นการเปรียบเทียบ new กับ make
 ```
var p *[]int = new([]int)       // allocates slice structure; *p == nil; rarely useful
var v  []int = make([]int, 100) // the slice v now refers to a new array of 100 ints

// Unnecessarily complex:
var p *[]int = new([]int)
*p = make([]int, 100, 100)

// Idiomatic:
v := make([]int, 100)
```

###Arrays
Arrays นั้นไม่ค่อยได้ใช้นักเพราะในภาษา Go เราจะใช้ Slices มากกว่า ความแตกต่างของ Arrays ใน Go กับ C คือ Arrays จะทำการ copy เสมอเมื่อมีการ assign ใส่ตัวแปรใหม่ หรือเมื่อใส่เข้าไปในฟังก์ชัน และขนาดของ Arrays ยังเป็นส่วนหนึ่งของ Type นั่นหมายความว่า [10]int กับ [20]int นั้นเป็นคนละ Type กัน

Arrays นั้นบางครั้งก็มีประโยชน์แต่ใช้ทรัพยากรสูงมาก ถ้าเราอยากให้ arrays แบบภาษา C ก็สามารถส่ง pointer ของ Arrays เข้าไปในฟังก์ชันแทนได้

###Slices
Slices นั้นครอบ Arrays ไว้อีกทีเพื่อให้ใช้งานได้ง่ายและทรงพลังกว่าสำหรับงานที่มีข้อมูลที่ต่อเนื่องกัน งานส่วนมากของ Go จะใช้ Slices นอกจากงานที่มีขนาดของมิติคงที่อย่าง Matrix

อย่างที่บอกไปว่า Slices นั้นชี้ไปที่ Arrays ถ้าเรา Assign ค่าของ Slice หนึ่งให้กับอีก Slice หนึ่งจะทำให้ Slice ทั้งสองชี้ไปที่เดียวกัน


```func (file *File) Read(buf []byte) (n int, err error)```
นี่คือตัวอย่างการรับ Slices เข้าไปในฟังก์ชัน
เราไม่ได้ส่งขนาดของมันเข้าไปด้วยเพราะฟังก์ชัน `len` จะบอกขนาดของมันได้อยู่แล้ว

ถ้า Slices นั้นมีขนาดใหญ่แต่เราต้องการส่งแค่ 32 ตัวแรกก็สามารถเรียกแบบนี้ได้ `n, err := f.Read(buf[0:32])` 

###Slices สองมิติ
เนื่องจาก Arrays และ Slices มีแค่มิติเดียว เราสามารถสร้างสิ่งที่เทียบเท่ากับ Slices หรือ Arrays สองมิติได้โดยการสร้าง Arrays of Arrays หรือ Slices of Slices
```
type Transform [3][3]float64  // A 3x3 array, really an array of arrays.
type LinesOfText [][]byte     // A slice of byte slices.
```

###Maps
Maps นั้นใช้เก็บค่าแบบ key-value โดย key จะเป็นประเภทอะไรก็ได้ที่สามารถทำ `equality operation` หรือสามารถเช็คการ `เท่ากับ` ได้ อย่างเช่น int, floting point, complex numbers, strings, pointers, interfaces, structs และ arrays ส่วน slices ไม่สามารถใช้เป็น key ได้เพราะไม่สามารถกำหนดการเท่ากับของมันได้

```
var timeZone = map[string]int{
    "UTC":  0*60*60,
    "EST": -5*60*60,
    "CST": -6*60*60,
    "MST": -7*60*60,
    "PST": -8*60*60,
}
```

การกำหนดค่าและดึงค่าของ Map นั้นสามารถทำเหมือนกับ Slices และ Arrays ได้แต่ key ไม่จำเป็นต้องเป็น integer การ return ถ้าเราดึงค่าจาก key ที่ไม่ได้มีใน Map เราจะได้ค่า 0 ของประเภทของ value หมายความว่าถ้า value เป็น boolean เราจะได้ค่า false แต่ถ้า value เป็น integer เราจะได้ค่า 0

```
attended := map[string]bool{
    "Ann": true,
    "Joe": true,
    ...
}

if attended[person] { // will be false if person is not in the map
    fmt.Println(person, "was at the meeting")
}
```

บางทีเราจำเป็นต้องแยกแยะระหว่างค่า 0 จากการที่ไม่มี key กับค่า 0 จริง ๆ ที่ได้จาก key
เราสามารถแยกแยะมันได้ด้วยการรับ multiple assignment อย่างเช่น

```
var seconds int
var ok bool
seconds, ok = timeZone[tz]
```

นี่เรียกว่า "comma ok" ซึ่ง ok จะเป็น false เมื่อไม่มี key นั้น
ด้านล่างคือตัวอย่างการใช้งานจริง

```
func offset(tz string) int {
    if seconds, ok := timeZone[tz]; ok {
        return seconds
    }
    log.Println("unknown time zone:", tz)
    return 0
}
```

ถ้าเราอยากเช็คแค่ว่า key นั้นมีจริงหรือเปล่า เราก็แค่ใช้ `_` เป็นตัวแปรแรก

```
_, present := timeZone[tz]
```

เราสามารถลบของมูลออกจาก map ได้ด้วยฟังก์ชัน `delete` (ใช้ได้แม้ว่า key นั้นจะไม่มีใน map)
```
delete(timeZone, "PDT")
```

###อื่นๆ
* Go มี Semicolon (;) ปิดท้าย แต่ไม่แสดงใน Code
