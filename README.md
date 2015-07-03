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

การใช้ new จะ Allocate Memory แต่ไม่ Initialize Memory (ให้ค่าเป็น 0) แล้ว return มาเป็น Address (Pointer)

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

### อื่นๆ
* Go มี Semicolon (;) ปิดท้าย แต่ไม่แสดงใน Code
