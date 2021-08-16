# Coffee Machine

Just enumerate all the stuff you have available (In eclipse ctrl+space go brr)
and search for interesting stuff, it's a jail escape after all, so once you know
the solution it's almost easy:

```java
import java.lang.invoke.*;
```

This package is basically a java.lang.reflect package but not blocked. So
you could just write into the constructor of the Main class in the root package:

```java
public Main() throws Throwable {
    MethodHandle x = MethodHandles.lookup().findStatic(Class.forName("uk.co.ractf.coffeemachine.kek.Other"), "getFlag", MethodType.methodType(String.class));
	System.out.println(x.invoke());
}
```

to invoke any public static function inside any package. Since we needed to come
from the coffeemachine package (or any subpackage) but not the two classes provided,
we needed to write our own, which for me was just the same, but instead of doing
a constructor we had to do a static function:

```java
public static String getFlag() {
		
		try {
			MethodHandle x = MethodHandles.lookup().findStatic(Class.forName("uk.co.ractf.coffeemachine.SecureStorage"), "getFlag", MethodType.methodType(String.class));
			
			for (StackTraceElement stackTraceElement : (new Throwable()).getStackTrace()) {
				System.out.println(stackTraceElement.getClassName());
			}
			
			System.out.println(x.invoke());
		} catch (Throwable e) {
			e.printStackTrace();
		}
		
		return "";
	}
```

(Note that it still returned string because at first I directly called the function
in the SecureStorage but that obviously didn't work, so I just did a proxy)
`ractf{j4v4_l4ng_inv0k3_api}`
