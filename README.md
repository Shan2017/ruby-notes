# ruby-notes

## 1. `method_missing` `respond_to_missing?` `const_missing`

```ruby
class MyObjectSystem < BasicObject
  DELEGATE = [:puts, :p]

  def method_missing(name, *args, &block)
    super unless DELEGATE.include? name
    ::Kernel.send(name, *args, &block)
  end

  def respond_to_missing?(name, include_private = false)
    DELEGATE.include?(name) or super
  end

  def self.const_missing(name)
    ::Object.const_get(name)
  end
end
```
