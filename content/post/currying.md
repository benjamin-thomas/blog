---
title: "Currying"
date: 2017-07-20T13:28:46+02:00
description: "Examples in ruby and javascript"
tags: [Ruby, Javascript]
---

## Ruby

```ruby
# =============================================================================
# First technique: define as a usual method, then convert to a lambda as needed
# =============================================================================

def mult(a,b)
  a * b
end

mult(3,4)       # => 12

method(:mult).  # returns a `Method` object, which behaves as a Proc (lambda)
  .call(3,4)

method(:mult)
  .curry        # returns the same Proc (lambda), and allows next partial application
  .call(3)      # apply the first partial transform

three_times = method(:mult).curry.(3)
three_times.(4) # => 12

# ==================================
# Second, more traditional technique
# ==================================

def mult2(a)
  lambda do |b|
    a * b
  end
end

alt_three_times = mult2(3)
alt_three_times.(4) # => 12
```

## Javascript

```javascript
function mult(a) {
    return function(b) {
        return a * b;
    }
}

var three_times = mult(3);
three_times(4) // => 12
```
