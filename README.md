# elixir-io_tty

Elixir module to handle key presses from the terminal in elixir.
It uses tty_sl to handle input.

THIS MODULE IS STILL IN DEVELOPMENT

## Usage

Add `:io_tty, git: "https://github.com/swalker90/elixir-io_tty.git" ` to your deps in mix.exs
For command line applications using escript add `emu_args: "-elixir ansi_enabled true -noinput"` to your escript definition.

To enable forward and back keys in your application configuration add `worker(IOTty, [])` to your application children and use IOTty.gets to get strings.
This will load default callbacks on each key to make it so that forward and back keys work.

## Callbacks

You may also define callbacks to be called instead of the default behaviour.
The callback map is formatted in the following way:
```
  %{
    :initial_state => {some_state_var, some_other_var},
    :default => &default_func/2,
    "\e[A" => &up_func/2,
  }
```

Each callback takes the key pressed and a state and returns the modified state.
State should contain anything to be held between key presses.
The initial state is determined by the `:initial_state` key in the callback map.
A callback can also return `{:stop_and_send, output}` which stops input and sends the variable back to the calling process.

For example, here are the default callbacks for handling printable characters and the return key:
```
  defp handle_ret("\r", {input, _}) do
    IO.write("\n\e[E")
    {:stop_and_send, input}
  end

  defp handle_char(<< b >>, {input, cursor}) when b in 32..126 do
    {pre, post} = cut(input, cursor)
    IO.write(<< b >> <> post <> back_up(post))
    {pre <> << b >> <> post, cursor+1}
  end
```

where the state is a tuple of `{input, cursor}` where `input` is the input characters so far and `cursor` is the cursor position.