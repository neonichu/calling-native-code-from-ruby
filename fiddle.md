# Calling native code from Ruby, no compiler required

![](images/ruby.jpg)

---

## We want to use some existing native library or system functionality

![](images/cogs.jpg)

---

## We don't want a C extension

![](images/8c6EBSz.gif)

---

## Because nobody got time to deal with user's compiler problems

![](images/tumblr_m0lwgmyRRh1r4zlnbo1_500.gif)

---

# Fiddle!

![](images/ruby.jpg)

---

## Somewhat exists in Ruby 1.9.x

![](images/WcCXCSZ.gif)

---

## But you really want 2.x

![](images/bE41G7O.gif)

---

## FFI for calling C from Ruby

![](images/azb6mBK_460sa_v1.gif)

---

## Uses dlopen / dlsym

![](images/cogs.jpg)

---

## Can also be used to call Objective-C 😱

![](images/dinosuar-t-rex.jpg)

---

```ruby
require 'fiddle'

image = Fiddle.dlopen(PATH)
function = Fiddle::Function.new(image[symbol], parameter_types, return_type)
result = function.call(a, b, c)
```

---

## Types

- Fiddle::TYPE_VOID
- Fiddle::TYPE_INT
- Fiddle::TYPE_CHAR
- Fiddle::TYPE_VOIDP
- ...

---

## Use `Fiddle::Handle.new` to refer to our own binary

---

### real-world example
## Xcodeproj

![](images/cocoapods-orange-on-grey.jpg)

---

### let's look at some code
## Using CoreFoundation

![](images/cocoapods-orange-on-grey.jpg)

---

### let's look at some code
## Using private Xcode functionality

![](images/cocoapods-orange-on-grey.jpg)

---

# Thanks

![](images/marin-6-thumb.gif)
