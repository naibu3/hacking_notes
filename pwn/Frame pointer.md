```c

#include <stdio.h>
#include <string.h>
#include <stdlib.h>

int limit, c;

int getebp () {
	__asm__("movl %ebp, %eax");
}

int proc(char *nombre)
}
	int *i;
	char buffer [256];
	i = (int *) getebp ();
	limit = i* - (int)buffer + 4;
	for (c = 0; c < limit && nombre[c] != '\0'; c++ )
		buffer[c] = nombre [c];

	printf("\nEncantado de conocerte: %s\n", buffer);
	return 0;
}

int main(int argc, char *argv[]) {

	if (argc < 2) {
		printf("\nUso: %s <nombre>\n", argv[0]);
		exit (0);
	}
proc (argv[1]);
return 0;
}

```