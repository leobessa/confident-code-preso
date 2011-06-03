!SLIDE
# Confident Code #

!SLIDE bullets
# Confident Code #
* Based on Avdi Grimm (@avdi) presentation 
* on RailsConf 2011
* by Leonardo Bessa (@leobessa)

!SLIDE bullets
# What is confident code? #
* The opposite of timid code.

!SLIDE subsection
# Timid Code #

!SLIDE title
# First, a story...

!SLIDE bullets
# Poor Storytelling #
* Tangents 
* Digressions 
* Lack of certainty 
* ...code can have these qualities as well.

!SLIDE smbullets
# Timid Code
* Lives in fear 
* Uncertain 
* Prone to digressions 
* Constantly second-guessing itself 
* Randomly mixes input gathering, error handling, and business logic 
* Imposes cognitive load on the reader

!SLIDE bullets
# Confident Code
* Says exactly what it intends to do 
* No provisos or digressions 
* Has a consistent narrative structure

!SLIDE bullets
# Source Code
* cowsay.rb 
* An interface to the "cowsay" program 
* http://github.com/avdi/cowsay

!SLIDE commandline
# Cowsay: ASCII art animals
    $ echo "Mooo" | cowsay
     ______
    < Mooo >
     ------
            \   ^__^
             \  (oo)\_______
                (__)\       )\/\
                    ||----w |
                    ||     ||

!SLIDE bullets
# Installing Cowsay
## Debian/Ubuntu

    $ sudo apt-get install cowsay

## Mac OS X 

    $ sudo port install cowsay
    or
    $ brew install cowsay

!SLIDE
# Cowsay.rb
    @@@ ruby
    require 'lib/cowsay'
    cow = Cowsay::Cow.new
    puts cow.say "Baaa"
    # >>  ______
    # >> < Baaa >
    # >>  ------
    # >>         \   ^__^
    # >>          \  (oo)\_______
    # >>             (__)\       )\/\
    # >>                 ||----w |
    # >>                 ||     ||

!SLIDE subsection
# Timid Code Structure

!SLIDE center
![timid-code-plain](/timid-code-plain.png)

!SLIDE center
![timid-code-annotated](/timid-code-annotated.png)

!SLIDE bullets
# The Narrative structure
* 1 Gather input
* 2 Perform work
* 3 Deliver results
* 4 handle failure

# **In that order! **

!SLIDE subsection
# Step 1: Gather Input

!SLIDE bullets
# To be confident, we must be sure of our inputs

!SLIDE bullets
# On Duck Typing
* True duck typing is a confident style 
* Duck typing doesn't ask "are you a duck?"
* `object.is_a?(String)`

!SLIDE bullets
# On Duck Typing
* Or even "can you quack?" 
* `object.respond_to?(:each)`

!SLIDE bullets
# On Duck Typing
* Treat the object like a duck 
* (If it isn't it will complain)

!SLIDE bullets
# The Switch (case) Smell
* Typecasing: Code that switches on the type of an object 
* The opposite of duck typing 
* A reaction to input variance 
* `nil` checks are the most common typecase 
* **It doesn't have to be this way**.

!SLIDE bullets
# Dealing with Uncertain Input
* Coerce 
* Reject 
* Ignore

!SLIDE bullets
# When in doubt, coerce
* Use `#to_s`, `#to_i`, `Array()` liberally 

!SLIDE bullets
# Ruby standard libs do this 
    @@@ ruby
    require 'pathname'
    path = Pathname("/home/avdi/.emacs")
    open(path) # => #<File:/home/avdi/.emacs>

!SLIDE bullets
# The "arrayification operator" 
    @@@ ruby
    Array([1,2,3])        # => [1, 2, 3]
    Array("foo")          # => ["foo"]
    Array(nil)            # => []

    # Gotcha:
    Array("foo\nbar")     # => ["foo\n", "bar"]

!SLIDE bullets
# Prefer `Array()` to `to_a`
    @@@ ruby
    Object.new.to_a
    # => [#<Object:0xb78a665c>]
    # !> default `to_a' will be obsolete

!SLIDE bullets
# Not using `Array()`
    @@@ ruby
    messages = case message
               when Array then message
               when nil then []
               else [message]
               end

!SLIDE bullets
# Using `Array()`
    @@@ ruby
    messages = Array(message)

!SLIDE bullets
# Gluing feathers to a pig
* When there is no consistent interface 
* Decorator Pattern 
* Wraps an object and adds a little extra

!SLIDE bullets
# Decorator Candidate
    @@@ ruby
    destination = 
      case options[:out]
      when nil  then "return value"
      when File then options[:out].path
      else options[:out].inspect # String?
      end

    @logger.info "Wrote to #{destination}"
    
!SLIDE title
#The WithPath Decorator
    @@@ ruby
    require 'delegate'
    
    class WithPath < SimpleDelegator
      def path
        case __getobj__
        when File then super
        when nil then "return value"
        else inspect
        end
      end
    end

!SLIDE title
#Using the Decorator
    @@@ ruby
    destination = WithPath.new(options[:out]).path
    # ...
    @logger.info "Wrote to #{destination}"

!SLIDE title
#Dynamically `#extend` objects
    @@@ ruby
    forrest_gump = Object.new
    
    module Runner
      def run
        "running"
      end
    end
    
    forrest_gump.extend(Runner)
    
    forrest_gump.run #=> "running"
    
!SLIDE title
#Dynamically add methods 
    @@@ ruby
    forrest_gump = Object.new
    
    def forrest_gump.run
       "running"
    end
    
    forrest_gump.run #=> "running"


!SLIDE bullets
# Reject Unexpected Values
* Not a duck. Not even a bird. 
* Will eventually cause an error 
* May not be clear where the error originated

!SLIDE bullets
# Assertive Code
* Confident Code asserts itself 
* Preconditions: state your demands up-front 
* Part of Design by Contract 
* Assertions don't have to be spelled "`assert()`"

<!-- !SLIDE title
## If code in one routine encounters an unexpected condition that it doesn't know how to handle, it throws an exception, essentially throwing up its hands and yelling 
## "I don't know what to do about this - I sure hope somebody else knows how to handle it!" 
###- Steve McConnell, Code Complete -->

!SLIDE title
# Basic Precondition
    @@@ ruby
    def say(message, options={})
      if options[:cowfile] && options[:cowfile] =~ /^\s*$/
        
        raise ArgumentError, "Cowfile cannot be blank"
        
      end
      # ...
      if options[:cowfile]
        command << " -f #{options[:cowfile]}"
      end
      # ...
    end

!SLIDE title
# Assertion Method
    @@@ ruby
    def assert(value, message="Assertion failed")
      raise Exception, message, caller unless value
    end

    # ...

    options[:cowfile] and
      assert(options[:cowfile].to_s !~ /^\s*$/)

!SLIDE subsection
# Assertion Notes

!SLIDE title
# `Exception` skips default `rescue` 
    @@@ ruby
    begin
      raise Exception, "Can't catch me!"
    rescue # rescues StandardError
      puts "Rescued!"
    end
    # ~> -:2: Can't catch me! (Exception)

!SLIDE title
# Third argument to `raise` sets stack trace 
    @@@ ruby
    raise Exception, message, caller

!SLIDE subsection
# Ignore Unnacceptable Values

!SLIDE bullets
# Guard Clause
    @@@ ruby
    def say(message, options={})
      return "" if message.nil?
      # ...
    end
* Short-circuit on nil message 
* Avoids special cases later in method

!SLIDE bullets
# Special Case Pattern
* Some arguments just **have** to be special 
* An OO way to deal with special cases 
* An object to represent the special case 
* Avoids `if`, `&&`, `try()`

!SLIDE code
# && and try()
    @@@ ruby
    # $? is either nil or a Process::Status object
    def check_child_exit_status!(status=$?)
      unless [0,172].include?(status && status.exitstatus)
        raise ArgumentError,
              "Command exited with status "\
              "#{status.try(exitstatus)}"
      end
    end

!SLIDE title
# Substitute Special Case for `nil`
    @@@ ruby
    # $? is either nil or a Process::Status object
    def check_child_exit_status!(status=$?)
    
      status ||= OpenStruct.new(:exitstatus => 0)
    
      unless [0,172].include?(status.exitstatus)
          raise ArgumentError,
                "Command exited with status"\
                "#{status.exitstatus}"
      end
    end

!SLIDE bullets
# Null Object
##"*A Null Object is an object with defined neutral ("null") behavior.*" 
###(Wikipedia)

!SLIDE bullets
# Null Object
* The special case value is usually `nil` 
* The special case for nil is usually "**do nothing**" 
* Responds to any message with nil (or itself) 
* Should be in the standard library

!SLIDE title
# A Basic Null Object
## Returns self from every call 
    @@@ ruby
    class NullObject
      def method_missing(*args, &block)
        self
      end

      def nil?; true; end
    end

    def Maybe(value)
      value.nil? ? NullObject.new : value
    end

!SLIDE bullets
# Nullifies chains of calls
    @@@ ruby
    foo = nil
    Maybe(foo).bar.baz + buz #=> #<NullObject:0x1019c5c50>

!SLIDE title
# NOT Using the Null Object
# if statement 
    @@@ ruby
    if options[:out]
      options[:out] << output
    end
    
!SLIDE title
# Using the Null Object
## Null Object
    @@@ ruby
    Maybe(options[:out]) << output
    
!SLIDE subsection
# Zero Tolerance for `nil`

!SLIDE bullets
# Zero Tolerance for `nil`
* `nil` is overused in Ruby code 
* It means too many things 
  * Error 
  * Missing Data 
  * Flag for "default behavior" 
  * Uninitialized variable 
  * Default return value for `if`, `unless`, empty methods. 
* **Nil checks are the most common form of timid code**

!SLIDE bullets
# Where'd that nil come from?!
    @@@ ruby
    post_id = params[:post_id]
    post    = if params[:blog]
                blog = get_blog(params[:blog])
                blog.posts.find(:first, post_id)
              end
    post.date     # =>
    # ~> -:4: undefined method `date' 
    #    for nil:NilClass (NoMethodError)
    

!SLIDE subsection
# Eliminating `nil`
## Kill it with fire.

!SLIDE title
# Use `Hash#fetch` when appropriate
    @@@ ruby
    collection.fetch(key) { fallback_action }

!SLIDE title
#  `#fetch` as an assertion
    @@@ ruby
    # Default error #
    opt = {}.fetch(:required_opt)
    # ~> -:1:in `fetch': key not found (IndexError)
    
    # Custom error #
    opt = {}.fetch(:required_opt) do
      raise ArgumentError, "Missing option!"
    end
    # ~> -:2: Missing option! (ArgumentError)

!SLIDE title
#  `#fetch` for defaults
    @@@ ruby
    # Using || 
    width = options[:width] || 40
    command << " -W #{width}"
    
    # Using #fetch 
    width = options.fetch(:width) {40}
    command << " -W #{width}"

* A more explicit default 
* One less conditional

!SLIDE title
# Don't use nil as a default
## Where did that nil come from? 
    @@@ ruby
    @logger = options[:logger]
    # ...
    @logger.info "Something happened"
    # ~> -:3: undefined method `info' for
    #    nil:NilClass (NoMethodError)
    
!SLIDE title
#Symbol Default
    @@@ ruby
    @logger = options.fetch(:logger){:no_logger_set}
    # ...
    @logger.info "Something happened"
    # ~> -:3: undefined method `info' for
    #    :no_logger_set:Symbol (NoMethodError)

!SLIDE title
#Null Object Default
    @@@ ruby
    def initialize(logger=NullObject.new)
      @logger = logger
    end

!SLIDE title
#Null Object with Origin
    @@@ ruby
    class NullObject
      def initialize
        @origin = caller.first
      end
      # ...
    end

!SLIDE
# Null Object with Origin
    @@@ ruby
    foo = NullObject.new
    bar = foo.bar
    baz = bar.baz
    # Now let's see where that nil originated
    baz
    #<NullObject:0x7fcace309800 @origin="(irb):68:in `new'">

!SLIDE subsection
# Step 2: Perform Work

!SLIDE bullets
# Perform Work
* Keep the focus on the work 
* Avoid digressions for error/input handling

!SLIDE bullets
# Conditionals for Business Logic
* Conditionals give you **more to think about**
* **Reserve conditionals for business logic**
* Minimize conditionals for error, input handling 
* Isolate error/input handling from program flow

!SLIDE bullets
# Conditionals
## Business logic
    @@@ ruby
    if post.published?
      # ...
    end
## Business logic?
    @@@ ruby
    if post
      # ...
    end

!SLIDE subsection
# Confident Styles of Work

!SLIDE title
# Confident Style: Chaining Work
    @@@ ruby
    def slug(text)
      Maybe(text).downcase.strip.tr_s('^a-z0-9', '-')
    end

    slug("Confident Code") # => "confident-code"
    slug(nil)              # => #<NullObject:0xb780863c>

!SLIDE title
# Confident Style: Iteration
## As exemplified by jQuery 
    @@@ javascript
    // From "jQuery in Action"
    $("div.notLongForThisWorld").fadeOut().
        addClass("removed");

!SLIDE bullets
# Confident Style: Iteration
* Single object operations are implicitly one-or-error 
* Iteration is implicitly 0-or-more 
* Chains of enumerable operations are self-nullifying 
* **`Cowsay#say` uses an iterative style...**

!SLIDE title
# Iterative Style
    @@@ ruby
    def say(message, options={})
      # ...
      messages = Array(message)
      message.map { |message|
        # ...
      }
      # ...
    end

    cow.say([]) # => ""

!SLIDE subsection
# Step 3: Deliver Results

!SLIDE bullets
# Don't return nil
* Help your callers be confident 
* Return a Special Case or Null Object 
* Or raise an error

!SLIDE subsection
# Step 4: Handle Failure

!SLIDE bullets
# Handle Failure
* Put the happy path first 
* Put error-handling at the end 
* Or in other methods.

!SLIDE title
# The Ruby Exception Handling Idiom
    @@@ ruby
    def foo
      # ...
    rescue => error # --- business/failure divider ---
      # ...
    end
## Neatly divides method into "work" and "error handling"

!SLIDE bullets
# Extract error handling methods
* Bouncer Method 
* Checked Method

!SLIDE bullets
# Bouncer Method
## Method whose job is to raise or do nothing 
    @@@ ruby
    def check_child_exit_status
      result = yield
      status = $? || OpenStruct.new(:exitstatus => 0)
      unless [0,172].include?(status.exitstatus)
        raise ArgumentError,
              "Command exited with status"\
              "#{status.exitstatus}"
      end
      result
    end

!SLIDE bullets
# Bouncer Method in Use
    @@@ ruby
    # Before:
    @io_class.popen(command, "w+") # ...
    # ...
    if $? && ![0,172].include?($?.exitstatus)
      raise ArgumentError,
            "Command exited with status #{$?.exitstatus.to_s}"
    end

    # After:
    check_child_exit_status {
      @io_class.popen(command, "w+") # ...
    }

!SLIDE code
## `begin rescue end`
    @@@ ruby
    @io_class.popen(command, "w+") do |process|
      results << begin
                   process.write(message)
                   process.close_write
                   result = process.read
                 rescue Errno::EPIPE
                   message
                 end
    end

!SLIDE title
# Checked Method
    @@@ ruby
    def checked_popen(command, mode, fail_action)
      check_child_exit_status do
        @io_class.popen(command, mode) do |process|
          yield(process)
        end
      end
    rescue Errno::EPIPE
      fail_action.call
    end

!SLIDE bullets
# Checked Method in Use
    @@@ ruby
    checked_popen(command,"w+", lambda{message}) do |process|
      # ...
    end

!SLIDE center
![confident-code-annotated](/confident-code-annotated.png)

!SLIDE bullets
# Observations on the Final Product
* It has a coherent narrative structure 
* It has lower complexity 
* It's not necessarily shorter

!SLIDE bullets
# Why Confident Code?
* Fewer paths == fewer bugs 
* Easier to debug 
* Self-documenting 
* "Write code for people first, the computer second"

!SLIDE smbullets
# Review
* A style, not a set of techniques 
* Consistent narrative structure 
* Handle input, perform work, deliver output, handle failure. 
* Avoid digressions 
* State your demands up front
* Terminate nils with extreme prejudice 
* Isolate error handling from the main flow

!SLIDE bullets
# Thank You
  