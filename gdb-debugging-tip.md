#GDB debugging tips

## Pop up GDB programmatically

```C
#include <signal.h>

/* The below pops up GDB. */
raise(SIGINT);
```

You will have to step twice to come
back out of the raise() call into your code.


# Assembly level debugging
