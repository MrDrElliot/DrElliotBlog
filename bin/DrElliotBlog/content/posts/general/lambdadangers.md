+++
title = "The dangers of Lambdas"
date = 2024-08-16T13:46:48-05:00
tags = ['Programming']
comments = true
+++

# Using Lambdas in Unreal can be dangerous!

---

Lambdas, or anonymous functions, in Unreal Engine can be very useful for writing concise and flexible code. However, they come with certain dangers, particularly when dealing with the lifetime of objects and capturing variables by reference. Here’s why:

### 1. **Capturing by Reference and Object Lifetimes**
   When you use lambdas in Unreal Engine, you might capture variables by reference, which is common for avoiding unnecessary copying. However, if the lambda is executed asynchronously (e.g., in a timer, or a delegate callback), the objects referenced might be destroyed or go out of scope by the time the lambda is executed. This leads to dereferencing invalid memory, causing crashes with errors like:

   ```
   Unhandled Exception: EXCEPTION_ACCESS_VIOLATION reading address 0x0000000000000000
   ```

   This particular error occurs because the lambda is trying to access an object or memory location that no longer exists, often leading to a null pointer dereference.

### 2. **Deferred Execution**
   In Unreal, many systems use deferred execution, where code is scheduled to run later (e.g., after a delay, or on the next frame). Lambdas are often used in these scenarios for their flexibility. However, if the lambda captures pointers or references to Unreal objects that may be garbage collected or destroyed before the lambda runs, the lambda will end up accessing invalid memory. This is especially problematic in cases where the object’s destruction isn’t under your control, such as when the object is managed by Unreal’s garbage collector.

### 3. **Complex Debugging**
   Lambdas can obscure the flow of your code, making debugging more complex. When a crash happens inside a lambda, the stack trace might not clearly indicate where or why it occurred. This is because the lambda could be executed in a completely different context from where it was defined, making it difficult to track down the source of the problem.

### 4. **Capturing `this`**
   Another common pitfall is capturing `this` inside a lambda. If the lambda is executed after the object has been destroyed, it will try to access a member function or variable of an object that no longer exists, leading to undefined behavior. This is particularly dangerous in Unreal, where object lifetimes are managed in complex ways, especially with asynchronous tasks.

### **How to Mitigate These Issues**

1. **Prefer Capturing by Value:** If possible, capture variables by value rather than by reference. This way, the lambda will have its own copy of the data, independent of the original object's lifetime.

2. **Use `TWeakObjectPtr`:** When capturing pointers to Unreal objects, consider using `TWeakObjectPtr` instead of raw pointers. `TWeakObjectPtr` will safely handle cases where the object is destroyed, allowing you to check if the object is still valid before accessing it.

3. **Check for Validity:** Before accessing any captured pointers or references, explicitly check their validity within the lambda. This includes checking if pointers are `nullptr` or if `TWeakObjectPtr` is still valid.

4. **Keep Lambdas Small and Focused:** Try to keep lambdas small and focused on a single task. This makes it easier to track down issues and reduces the likelihood of capturing unnecessary variables.

5. **Using a `Weak Lambda`.** Some may consider this a hack, but a *weak lambda* allows you to safely reference an object inside of a lambda, and avoid calling the lambda through the delegate if the owning object is invalid.
-- Here is a quick example of a weak lambda in use. Taken from `TestBTTask_TimerBasedLatent.cpp`

```cpp
	if (const UWorld* World = OwnerComp.GetWorld())
	{
		const float DeltaTime = NumTicksExecuting * FAITestHelpers::TickInterval;
		World->GetTimerManager().SetTimer(TaskMemory->TimerHandle,
			FTimerDelegate::CreateWeakLambda(this, [&OwnerComp, TaskMemory, this]() // <---- This is the declaration of a weak lambda
			{
				TaskMemory->TimerHandle.Invalidate();

				ensure(!TaskMemory->bIsAborting);
				LogExecution(OwnerComp, LogIndexExecuteFinish);
				FinishLatentTask(OwnerComp, LogResult);
			}), DeltaTime, false);
	}
```

By being aware of these potential dangers and taking appropriate precautions, you can safely use lambdas in Unreal Engine without running into unexpected crashes or undefined behavior.
