---
title: "Closures"
date: 2017-07-20T12:18:25+02:00
description: "Examples in ruby and javascript"
draft: false
---

## Ruby


```ruby
def cycle(collection)
  index = -1
  lambda do
    index += 1
    collection[index % collection.size]
  end
end

c = cycle(['red', 'green', 'blue'])

4.times.map do
  c.call
end # => => ["red", "green", "blue", "red"]
```

## Javascript

```javascript
function cycle(coll) {
    var idx = -1;
    return function() {
        idx += 1;
        return coll[idx % coll.length];
    }
}

c = cycle(['red', 'green', 'blue'])
Array(4).fill(null).map(function() {
    return c();
}) // => ["red", "green", "blue", "red"]
```
