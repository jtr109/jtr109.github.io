---
title: Be Careful When Defining Singleton Using Double-Checked Locking
tags:
  - CSharp
  - DCL
  - Singleton
date: 2024-03-28 22:17:07
---


Initializing instance must be an atomic method. Otherwise, there should be some unexpected behaviors.

For example, let's say that we are going to define a singleton which contains a property in int type with 1 as value. If the singleton is initialized, reuse the instance of the singleton.

```cs
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

public sealed class Singleton
{
    private static Singleton _instance;
    private static readonly object LockObject = new object();
    private int _number;

    private Singleton()
    {
    }

    public static Singleton Instance
    {
        get
        {
            if (_instance == null)
            {
                lock (LockObject)
                {
                    if (_instance == null)
                    {
                        _instance = new Singleton();
                        Task.Delay(100).Wait(); // Sleep for 10 ms
                        _instance._number= 1; // Set number as 1
                    }
                }
            }
            return _instance;
        }
    }

    public int Number => _number;
}

class Program
{
    static void Main()
    {
        // Create multiple tasks to simulate concurrent access
        var tasks = new List<Task>();
        for (int i = 0; i < 10; i++)
        {
            tasks.Add(Task.Run(() =>
            {
                var singleton = Singleton.Instance;
                Console.WriteLine($"Thread {Task.CurrentId}: Number count = {singleton.Number}");
            }));
        }

        Task.WaitAll(tasks.ToArray());
        
        var singleton = Singleton.Instance;
        Console.WriteLine($"Finally: Number count = {singleton.Number}");
    }
}
```

If we define the `Singleton` class like above, what will be shown in the console?

A:

```plain
Thread 1: Number count = 0
Thread 2: Number count = 0
Thread 3: Number count = 0
Thread 4: Number count = 0
Thread 5: Number count = 0
Thread 6: Number count = 0
Thread 9: Number count = 1
Thread 10: Number count = 1
Thread 7: Number count = 1
Thread 8: Number count = 1
Finally: Number count = 1
```

B:

```plain
Thread 1: Number count = 1
Thread 2: Number count = 1
Thread 3: Number count = 1
Thread 4: Number count = 1
Thread 5: Number count = 1
Thread 6: Number count = 1
Thread 9: Number count = 1
Thread 10: Number count = 1
Thread 7: Number count = 1
Thread 8: Number count = 1
Finally: Number count = 1
```

Unfortunately, the answer is **A**. As an explanation, when one thread initialize the instance by `_instance = new Singleton();`, one another code find the instance is not null and expect the `Number` has been set to 1. But actually, the value of `Number` property is not set yet. Then, the DCL (double-checked locking) doesn't achieve the target it is used in this case.
