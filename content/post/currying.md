---
title: "Currying"
date: 2017-07-20T13:28:46+02:00
description: "Examples in ruby and javascript"
draft: false
---

## Ruby

```ruby
# First method, define as a usual method, and convert to a lambda as needed

def mult(a,b)
  a * b
end

mult(3,4) # => 12

method(:mult). # returns a `Method` object, which behaves as a Proc (lambda)
  .call(3,4)

method(:mult)
  .curry # returns the same Proc (lambda), and allows next partial application
  .call(3) # apply the first partial transform

thirds = method(:mult).curry.(3)
thirds.(4) # => 12

def mult2(a) #test
  lambda do |b|
    a * b
  end
end

# Second, more traditional method

thirds2 = mult2(3)
thirds2.(4) # => 12
```

## Javascript

```javascript
function mult(a) {
    return function(b) {
        return a * b;
    }
}

var thirds = mult(3);
thirds(4) // => 12
```
