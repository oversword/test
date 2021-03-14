# Test - MineTest
> A test suite for MineTest, allows you to test the functional components of your mod

## Integrating the Mod

For ease of integration, you should add this mod as an optional dependency, and run the tests on the condition that the mod is enabled. That will allow you to include your test file in your `init.lua` without having to comment and uncomment it to enable your tests, just enable or disable the mod.

To achieve this:

Put this at the top of your test file so that it will only run when the mod is enabled
```lua
if not minetest.global_exists('test') then return end
```
Add this mod as an optional dependency to you mod in `mod.conf`
```conf
optional_depends = test
```
Or, in `depends.txt`
```txt
test?
```

## Using the Mod

Typically, it's cleanest to import the abilities you will use as local vars at the top of your test file for easier calling, for instance:
```lua
local describe = test.describe
local it = test.it
local stub = test.stub
```


### Do It!
To run the tests you must call the `execute` function after all your tests have been defined
```
execute()
```

### Describe
You can use any number of nested `describe` calls as you like, to describe the different levels of your mod's functionality:
```lua
describe("My Mod", function ()
	describe("Major Component", function ()
		describe("Component's Feature", function ()
		end)
		describe("Minor Sub-Component", function ()
			describe("Sub-Component's Feature", function ()
			end)
		end)
	end)
end)
```

### It
You can only use one level of `it` calls, nesting them will not do anything useful. `it` must be inside a `describe`
```lua
describe("My Mod", function ()
	it("does something", function ()
		-- this is fine
	end)
	describe("Major Component", function ()
		it("does something else", function ()
			-- so is this

			--[[ THIS IS NOT
			it("doesn't do this", function ()
				-- Don't do this
			end)
			]]
		end)
	end)
end)
```

### Asserting Equality
You can assert that two values are equal or not equal using the two assertion methods. This will deep-match tables by value, as well as all other data types.
The message should be the condition that's being tested, e.g. `"Color should be blue"`
```lua
assert.equal( expected, received, message )
assert.not_equal( expected, received, message )
```

### Expecting Errors
You can assert that the test should throw an error, this must be called before the function that is expected to throw an error. Only one error can be expected during an `it` call, as the error will exit the function.
The message will match error messages that include the string you have provided, you do not need to provide the entire error message. You can also provide no message to simply expect an error to occur, without specifying exactly which one.
```lua
expect.error( message )
```

### Test Sequence Callbacks
All of these must be called before the `it` tests for the time being, this should be considered a bug.

```lua
-- Will be called after each `it` test has executed
after_each
-- Will be called before each `it` test has executed
before_each

-- Will be called after all the `it` tests have been executed in this `describe` definition
after_all

-- Will be called before any `it` tests have been executed in this `describe` definition
before_all
```

### Stub
You can create stubs that will record when they are called, however there is no magic and you will need to override and replace globals yourself
```lua
describe("My Mod", function ()

	-- Save the original
	local original_global = minetest.function_to_stub

	-- Create a stub and override a global function with stub().call
	local function_stub = stub()
	minetest.function_to_stub = funcition_stub.call

	-- Replace the original once everything is done
	after_all(function ()
		minetest.function_to_stub = original_global
	end)

	it("does something", function ()

		-- Call the stub or a function that will call it
		minetest.function_to_stub()

		-- Assert that it's been called
		function_stub.called_times(1)

	end)
end)
```

You can also pass a mock function into this stub that will be called in place of the original
```lua
local function_stub = stub(function (some, args...)
	return "mocked value"
end)
```
#### Public Stub API

The table returned by the `stub()` call provides some public methods
```lua
-- Call the stub function itself
stub.call( some, args... )


-- Asserts that the stub function has been called this many times at the point of execution
-- Must be called after the stub function has
stub.called_times( occurrences )

-- Asserts that the stub function has been called with the given arguments at the point of execution
-- Must be called after the stub function has
stub.called_with( some, args... )


-- Asserts that the stub function will have been called this many times after everything has finished executing
-- Can be called at any time in the `it` definition
stub.to_be_called_times( occurrences )

-- Asserts that the stub function will have been called with the given arguments after everything has finished executing
-- Can be called at any time in the `it` definition
stub.to_be_called_with( some, args... )

-- Asserts that the stub function will be called in the given number of seconds after the call is made
-- Can be called at any time in the `it` definition
stub.to_be_called_in( seconds )

-- Asserts that the stub function will be called in the given number of seconds after the call is made, with the arguments given
-- Can be called at any time in the `it` definition
stub.to_be_called_in_with( seconds, some, args... )
```

