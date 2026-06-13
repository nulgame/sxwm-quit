# sxwm-quit 

sxwm window manager for xorg does not prevent you from exiting the session without confirmation.

So I prepared a small patch for the sxwm.c file so that when using the rofi program, the window manager would ask the user to confirm this action.

## Dependency

rofi have to be installed on your OS.

## Patch usage

Copy patch file into sxwm/src/ directory and then do following command
```bash
patch sxwm.c < confirmedquit.patch
```

## Result

After patching sxwm.c quit function will looks as follow:

```c
void quit(void)
{
  FILE *cq = popen(
      "echo \"NO\nYES\" | rofi -dmenu -i -p \"Quit window manager? (or ESC) \" -theme-str 'window { width:60%; height:30%; border-color: rgba (255,0,0,100%);}'"
      , "r");
  if (cq != NULL) {
    char buf[1024];
    if (fgets(buf, sizeof(buf), cq) == NULL) {
      fprintf(stderr, "sxwm: quit() -> error reading pipe!\n");
      return;
    }
    pclose(cq);
    if (strcmp(buf, "YES\n") == 0) {

      XSync(dpy, False);
      XFreeCursor(dpy, cursor_move);
      XFreeCursor(dpy, cursor_normal);
      XFreeCursor(dpy, cursor_resize);
      XCloseDisplay(dpy);
      puts("quitting...");
      running = False;
    }
  }
  return;
}

```
