# Lino

Command line execution utilities.

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'lino'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install lino

## Usage

Lino allows commands to be built and executed:

```ruby
require 'lino'
  
command_line = Lino::CommandLineBuilder.for_command('ruby')
    .with_flag('-v')
    .with_option('-e', 'puts "Hello"')
    .build
    
puts command_line.to_s 
# => ruby -v -e puts "Hello"
  
command_line.execute 
# ruby 2.3.1p112 (2016-04-26 revision 54768) [x86_64-darwin15]
# Hello
```

### `Lino::CommandLineBuilder`

The `CommandLineBuilder` allows a number of different styles of commands to be 
built:

```ruby
# commands with flags
Lino::CommandLineBuilder.for_command('ls')
    .with_flag('-l')
    .with_flag('-a')
    .build
    .to_s

# => ls -l -a

# commands with options
Lino::CommandLineBuilder.for_command('gpg')
    .with_option('--recipient', 'tobyclemson@gmail.com')
    .with_option('--sign', './doc.txt')
    .build
    .to_s

# => gpg --recipient tobyclemson@gmail.com --sign ./doc.txt

# commands with an option repeated multiple times
Lino::CommandLineBuilder.for_command('example.sh')
    .with_repeated_option('--opt', ['file1.txt', nil, '', 'file2.txt'])
    .build
    .to_s

# => example.sh --opt file1.txt --opt file2.txt

# commands with arguments 
Lino::CommandLineBuilder.for_command('diff')
    .with_argument('./file1.txt')
    .with_argument('./file2.txt')
    .build
    .to_s

# => diff ./file1.txt ./file2.txt

# commands with an array of arguments
Lino::CommandLineBuilder.for_command('diff')
    .with_arguments(['./file1.txt', nil, '', './file2.txt'])
    .build
    .to_s

# => diff ./file1.txt ./file2.txt

# commands with custom option separator
Lino::CommandLineBuilder.for_command('java')
    .with_option_separator(':')
    .with_option('-splash', './images/splash.jpg')
    .with_argument('./application.jar')
    .build
    .to_s

# => java -splash:./images/splash.jpg ./application.jar

# commands using a subcommand style
Lino::CommandLineBuilder.for_command('git')
    .with_flag('--no-pager')
    .with_subcommand('log') do |sub|
      sub.with_option('--since', '2016-01-01')
    end
    .build
    .to_s

# => git --no-pager log --since 2016-01-01

# commands with multiple levels of subcommand
Lino::CommandLineBuilder.for_command('gcloud')
    .with_subcommand('sql')
    .with_subcommand('instances')
    .with_subcommand('set-root-password')
    .with_subcommand('some-database') do |sub|
      sub.with_option('--password', 'super-secure')
    end
    .build
    .to_s
    
# => gcloud sql instances set-root-password some-database --password super-secure

# ... or alternatively
Lino::CommandLineBuilder.for_command('gcloud')
    .with_subcommands(
      %w[sql instances set-root-password some-database]
    ) do |sub|
      sub.with_option('--password', 'super-secure')
    end
    .build
    .to_s
    
# => gcloud sql instances set-root-password some-database --password super-secure
  
# commands controlled by environment variables
Lino::CommandLineBuilder.for_command('node')
    .with_environment_variable('PORT', '3030')
    .with_environment_variable('LOG_LEVEL', 'debug')
    .with_argument('./server.js')
    .build
    .to_s
    
# => PORT=3030 LOG_LEVEL=debug node ./server.js
```

### `Lino::CommandLine`

A `CommandLine` can be executed using the `#execute` method:

```ruby
command_line = Lino::CommandLineBuilder.for_command('ls')
    .with_flag('-l')
    .with_flag('-a')
    .with_argument('/')
    .build
    
command_line.execute
  
# => <contents of / directory> 
```

By default, the standard input stream is empty and the process writes to the 
standard output and error streams.

To populate standard input:

```ruby
command_line.execute(stdin: 'something to be passed to standard input')
```

The `stdin` option supports any object that responds to `each`, `read` or 
`to_s`.

To provide custom streams for standard output or standard error:

```ruby
require 'stringio'
  
stdout = StringIO.new
stderr = StringIO.new
  
command_line.execute(stdout: stdout, stderr: stderr)
  
puts "[output: #{stdout.string}, error: #{stderr.string}]"
```

The `stdout` and `stderr` options support any object that responds to `<<`.

## Development

To install dependencies and run the build, run the pre-commit build:

```shell script
./go
```

This runs all unit tests and other checks including coverage and code linting / 
formatting.

To run only the unit tests, including coverage:

```shell script
./go test:unit
```

To attempt to fix any code linting / formatting issues:

```shell script
./go library:fix
```

To check for code linting / formatting issues without fixing:

```shell script
./go library:check
```

You can also run `bin/console` for an interactive prompt that will allow you to 
experiment.

## Contributing

Bug reports and pull requests are welcome on GitHub at 
https://github.com/infrablocks/lino. This project is intended to be a safe, 
welcoming space for collaboration, and contributors are expected to adhere to 
the [Contributor Covenant](http://contributor-covenant.org) code of conduct.

## License

The gem is available as open source under the terms of the 
[MIT License](http://opensource.org/licenses/MIT).
