# Modifying the JVM Runtime

## Needed before

Make sure you have the JDK and JRE installed for compiling and running .jar files.

## Purpose
The purpose of modifying the JVM Runtime is to add basic HTTP Server and Client capabilities to Virgil. I wish this to be the basis for another project related to an LSP for Virgil integrated in VSCode. (We'll have to see if anything materializes though). Virgil is a really great and fun language, but it lacks autocomplete, and sometimes manually going up classes without any help at all from the IDE is a bit harsh.

The reasoning behind of why we should modify it only for JVM is simple: VSCode runs on all 3 major operating systems (Linux, Windows, OS X) and JVM does too. Implementing HTTP Servers for all 3 platforms does not look feasible to me (at the time of writing, Virgil for OS X on M chips - ARM - is not supported, nor is it for any other ARM architecture).

Since through JVM we can execute Java code directly, and not just that, but we can choose from a handful of Java packages to interact with and make our work easier, this seems like a perfect quick and small project.

## Basic tutorial for modifying the System class
Virgil is mostly written in virgil, the only time we can actually play with Java is by playing with the `rt/jvm/V3S_System.java` file.

Let's see how we'd do this by adding a simple `fooBar` method attached to the `System` class and have it print `fooBar` when called.

1. We modify the `rt/src/V3S_System.java`:

```diff
public class V3S_System {
    ...

+    public static void fooBar() {
+        System.out.println("fooBar");
+    }

    ...
}
```

2. We run the `Makefile` from the same directory: 
```sh
> make
```

3. Next, we have to modify the signature of the System class, so we don't get errors when using our new method in programs. This can be done pretty easily by modifying the `aeneas/src/v3/SystemModule.v3` file:

```diff
enum SystemCall(eval: (SystemCallState, Arguments) -> Result, paramTypes: Array<Type>, returnType: Type) {
    ...
    ticksNs(	SystemCallState.ticksNs,	TypeUtil.NO_TYPES, Int.TYPE),
+   fooBar(		SystemCallState.fooBar,		TypeUtil.NO_TYPES, Void.TYPE),
}

class SystemCallState {
    ...

+   def fooBar(args: Arguments) -> Result {
+		return Values.BOTTOM;
+	}

    ...
}
```

4. Next, we regenerate our compiler (run the command in the base directory): 
```sh
> make bootstrap
```

4. Next, we can just write our own program:

```
// 1.v3
def main() {
    System.fooBar();
}
```

5. We then have to compile it into a `.jar`: 
```sh
> v3c-jar 1.v3
```

6. Finally, we just execute it with java: 
```sh
> java -jar 1.v3
```