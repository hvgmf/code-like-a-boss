 
# How to write maintainable code

Writing maintainable code in a big code base that is human friendly is hard. These are good guidlines to follow but not a gospel of the ultimate truth, there will always be exceptions.

## General

- Avoid nested if statements
- Avoid negations
- Favor early returns
- Avoid using else
- Don't else after return
- Avoid complex expressions
- Don't double assign variables
- Don't use global variables
- Avoid comments in code
- Don't use C style comments for documentation
- Use clear and obvious names for identifiers
- Avoid magic strings and numbers

### Avoid nested if statements

Nesting makes the logic more complex and the code harder to understand, use only in the rare cases when absolutely necessary.

❌ NO
```go
    func test(x int) {
        if (x > 0) {
            dostuff1()
            if (x > 10) {
                dostuff2()
            }
        }
    }
```

✅ YES
```go
    func test(x int) {
        if (x > 0) {
            dostuff1()
        }
        if (x > 10) {
            dostuff2()
        }
    }
```

### Avoid negations
"if is not equals" is more complicated to read than "if equals".

❌ NO
```go
    if !user.isLoggedIn() {
        login()
    } else {
        logout()
    }
```

✅ YES
```go
    if user.isLoggedIn() {
        logout()
    } else {
        login()
    }
```

### Favor early returns
Terminating as soon as possible makes the code easier to understand.

❌ NO
```go
    func test(x int) {

        dostuff1()

        if (x != 0) {   // does something happen after this if x is 0?
            dostuff2()
            dostuff3()
            ...
        }
    }
```
✅ YES
```go
    func test(x int) {

        dostuff1()

        if (x == 0) {  // we know only dostuff1() is called if x is 0
            return
        }

        dostuff2()
        dostuff3()
        ...
    }
```

### Avoid using else
Branching with else is often not necessary and will needlessly complicate the code most of the time.

❌ NO
```go
    func test(x int, y int, z int) {

        if (x > 0) {
            return x + z
        } else {
            if (y > 0) {
                return y + z
            }
        }

        return z
    }
```

✅ YES
```go
    func test(x int, y int, z int) {

        if (x > 0) {
            return x + z
        }        

        if (y > 0) {
            return y + z
        }

        return z
    }
```

### Don't else after return
It's extra code that branch into a pointless new variable scope.

❌ NO
```go
    func test(x int, y int) {
        if (x > 0) {
            return x
        } else {
            return y
        }
    }
```

✅ YES
```go
    func test(x int, y int) {
        if (x > 0) {
            return x
        }
        return y 
    }
```

### Avoid complex expressions
Split up complex expressions for readability, and invert the logic if needed.

❌ NO *- This is way too hard to read, and we even need a comment to explain its purpose*
```go
    func test(x int, int y) {
        // check if we need to open the pod doors
        if (x != y && (x < 0 || y != 10) && (y > 0 || x != 10)) {  
            openPodDoors()
        }
    }
```

✅ YES *- Much better and now the code reads like a sentence, no comment necessary*
```go
    func test(x int, y int) {
        if (shouldOpenPodDoors(x, y)) { 
            openPodDoors()
        }
    }

    func shouldOpenPodDoors(x int, y int) bool {
        if x == y {
            return false
        }
        if x >= 0 && y == 10 {
            return false
        }
        if y <= 0 && x == 10 {
            return false
        }
        return true
    }
```

### Don't double assign variables
Don't re-assign a value to a variable unless its previous value has been used in some capacity.

❌ NO
```go
    config := getProductionValues();

    if (environment == Development) {
        config = getDevelopmentValues()
    }
```

✅ YES
```go
    var config Configuration

    if (environment == Development) {
        config = getDevelopmentValues()
    } else {
        config = getProductionValues()
    }
```

❌ NO *- unclear intent, x is declared and assigned a value that has no purpose*
```go
    x := 0 

    if (test()) {
        x = 10
    } else {
        x = 20
    }
```

✅ YES *- clear intent, x is declared and will be assigned a value later*
```go
    var x int

    if (test()) {
        x = 10
    } else {
        x = 20
    }
```

### Don't use global variables

They make the code hard/impossible to unit test and are just bad design in general in large applications.

❌ NO
```go
    var database Database

    func update(data string) {
       database.write(data)
    }
```
✅ YES
```go
    func update(database Database, data string) {
       database.write(data)
    }
```

### Avoid comments in code
Write comments only when necessary to explain reason and context, not function or as an excuse to write code with unclear intent.

❌ NO
```go
    // set speed to 88 mph
    var speed = 88
```

✅ YES
```go
    // 88 mph is needed to go back in time
    var speed = 88
```

### Don't use C style comments for documentation
Use C++ style comments to write permanent comments, and C style comments when you need to temporarily disable large blocks of code.

❌ NO *- This comment will interfere with any overarching /\* \*/ later*
```go
    /* This code does things */
    func things() {
        ...
    }
```

✅ YES *- This comment will not block if we need to disable parts of the code with /\* \*/ later*
```go
    // This code does things
    func things() {
        ...
    }
```

### Use clear and obvious names for identifiers
Name things after what they actaully do and don't abbreviate.

❌ NO
```go
    func update(s string, i int, img []byte) {
        ...
    }
```
✅ YES
```go
    func updateProfile(name string, age int, photo []byte) {
        ...
    }
```

### Avoid magic strings and numbers
Use constants to make the code more readable and avoid human errors.

❌ NO
```go
    if (permission == "is_adimn") {
        admin()
    }

    if (status == 401) {
        denied()
    }
```

✅ YES
```go
    const ADMIN = "is_admin"
...
    if (permission == ADMIN) {
        admin()
    }

    if (status == http.StatusUnauthorized) {
        denied()
    }
```

## Go specific

- Avoid writing literal json

### Avoid writing literal json
it's both ugly, hard to read, and very easy to get wrong.

❌ NO
```go
    text := `{"name":`+name+`,`"user":`,+username+"}"
```

✅ YES
```go
    data, _ := json.Marshal(struct {
        Name string
        User string
    }{
        Name: name,
        User: username,
    })

    text = string(data)
```
