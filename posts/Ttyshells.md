---
layout: default
title : muzec - Spawning A TTY Shell, Cheat Sheets
---

Getting A Dumb Shell Can Be Annoying At Times During A Pen Tests You May Obtain A Shell Without Having tty, Yet Wish To Enumerate The System Further. Here Are Some Commands Which I Always Use To Spawn A TTY Shell.

Cheat Sheets For Spawning A TTY Shell.

```python -c 'import pty; pty.spawn("/bin/sh")'```

```python -c 'import pty; pty.spawn("/bin/bash")'```

```python3 -c 'import pty; pty.spawn("/bin/sh")'```

```python3 -c 'import pty; pty.spawn("/bin/bash")'```

```/bin/bash -i```

```/bin/sh -i```

```echo os.system('/bin/bash')```

```perl —e 'exec "/bin/sh";'```

```perl: exec "/bin/sh";```

```ruby: exec "/bin/sh"```

```lua: os.execute('/bin/sh')```

Bypassing Shell Restriction And Spawning A TTY Shell.

Within Vi

```:set shell=/bin/bash:shell```

Within Vi

```:!bash```

Within Nmap

```
nmap --interactive

!sh
```

Within IRB

```exec "/bin/sh"```

Greeting From [Muzec](https://twitter.com/muzec_saminu)

<br> <br>
[Back To Home](../index.md)
<br>
