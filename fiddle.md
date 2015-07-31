# Calling native code from Ruby

## eurucamp 2015

### Boris Bügling - @NeoNacho

![20%, original, inline](images/contentful.png)

![](images/ruby.jpg)

<!-- Use next/black as theme -->

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

# Xcodeproj

- Part of CocoaPods
- Interacts with Xcode projects
- Used to include a C extension

![](images/Xcode.png)

---

# Xcodeproj issues

```bash
$ bundle exec rake
...
linking shared-object xcodeproj_ext.bundle
clang: error: unknown argument: '-multiply_definedsuppress'
[-Wunused-command-line-argument-hard-error-in-future]
clang: note: this will be a hard error (cannot be
downgraded to a warning) in the future
make: *** [xcodeproj_ext.bundle] Error 1
```

![](images/Xcode.png)

---

# Xcodeproj issues

```bash
$ gem install xcodeproj 
Building native extensions.  This could take a while...
ERROR:  Error installing xcodeproj:
    ERROR: Failed to build gem native extension.

        /Users/siuying/.rvm/rubies/ruby-1.9.3-p194/bin/ruby extconf.rb
checking for -std=c99 option to compiler... *** extconf.rb failed ***
Could not create Makefile due to some reason, probably lack of
necessary libraries and/or headers.  Check the mkmf.log file for more
details.  You may need configuration options.
```

![](images/Xcode.png)

---

# Xcodeproj issues

```bash
creating Makefile
make
[...]
make install
/usr/bin/install -c -m 0755 xcodeproj_ext.bundle ./.gem.20130406-29555-zd0wy8
ERROR:  While executing gem ... (NoMethodError)
    undefined method `join' for nil:NilClass
```

![](images/Xcode.png)

---

![inline](images/nokogiri.png)

![](images/XML.jpg)

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
function = Fiddle::Function.new(image[symbol], 
								parameter_types,
								return_type)
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

```ruby
def self.extern_image(image, symbol, parameter_types, return_type)
  symbol = symbol.to_s
  create_function = symbol.include?('Create')
  function_cache_key = "@__#{symbol}__"

  define_singleton_method(symbol) do |*args|
    unless function = instance_variable_get(function_cache_key)
      function = Fiddle::Function.new(image[symbol],
                                      parameter_types,
                                      return_type)
      instance_variable_set(function_cache_key, function)
    end

    result = function.call(*args)
    create_function ? CFAutoRelease(result) : result
  end
end
```

---

```ruby
PATH = '/System/Library/Frameworks/CoreFoundation.framework/CoreFoundation'

def self.image
  @image ||= Fiddle.dlopen(PATH)
end

def self.extern(symbol, parameter_types, return_type)
  extern_image(image, symbol, parameter_types, return_type)
end

extern :CFDataGetLength, [CFTypeRef], CFIndex
extern :CFDataGetBytePtr, [CFTypeRef], VoidPointer

def self.CFStringToRubyString(string)
  data = CFStringCreateExternalRepresentation(NULL,
                                            string,
                                            KCFStringEncodingUTF8,
                                            0)

  if data.null?
    raise TypeError, 'Unable to convert CFStringRef.'
  end

  bytes_ptr = CFDataGetBytePtr(data)
  result = bytes_ptr.to_str(CFDataGetLength(data))
  result.force_encoding(Encoding::UTF_8)
  result
end
```

---

### let's look at some code
## Using private Xcode functionality

![](images/cocoapods-orange-on-grey.jpg)

---

```ruby
def self.objc_msgSend(args, return_type = CoreFoundation::VoidPointer)
  arguments = [CoreFoundation::VoidPointer, CoreFoundation::VoidPointer] + args

  Fiddle::Function.new(image['objc_msgSend'], arguments, return_type)
end

class CFDictionary < NSObject
  public

  def initialize(dictionary)
    @dictionary = dictionary
  end

  def plistDescriptionUTF8Data
    selector = 'plistDescriptionUTF8Data'
    return nil unless NSObject.respondsToSelector(@dictionary, selector)

    plistDescriptionUTF8Data = CFDictionary.objc_msgSend([])
    plistDescriptionUTF8Data.call(
      @dictionary,
      CoreFoundation.NSSelectorFromString(CoreFoundation.RubyStringToCFString(selector)))
  end
end
```

---

# Recap

- C extensions have a bad UX
- Fiddle provides a way to use native code dynamically
- Which eliminates all the compilation hassle
- We can even call Objective-C if we want

![](images/ruby.jpg)

---

# @NeoNacho

boris@contentful.com

http://buegling.com/talks

![](images/contentful-bg.png)
