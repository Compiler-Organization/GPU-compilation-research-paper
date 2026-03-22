# GPU-compilation-research-paper
A research paper on compiling shaders to be used for the GPU, but with conditional branching.

# Problems
* Parallel execution
  * Since each "instruction" would be handled on each thread, this not allow conditional branching, variable assignment and such.
* Mutability
  * Passing variables between threads would not be possible / it would be slow.
  * Mutable variables would not be thread safe.

# Proposed solution
We would need an instruction set that would be executed on each seperate thread/core on the GPU, like assembly.
To branch code, we would need to either create new threads, or reuse threads to execute the code block of said branching statement.

```asm
mov eax, 10
add eax, 10
ret ; 20
```

Would (directly translated, using ComputeSharp) be

```cs
internal class Program
{
    static void Main(string[] args)
    {
        int[] array = { 10 };

        using ReadWriteBuffer<int> buffer = GraphicsDevice.GetDefault().AllocateReadWriteBuffer(array);
        GraphicsDevice.GetDefault().For(buffer.Length, new MultiplyByTwo(buffer));

        buffer.CopyTo(array);
    }
}

[ThreadGroupSize(DefaultThreadGroupSizes.X)]
[GeneratedComputeShaderDescriptor]
public readonly partial struct MultiplyByTwo(ReadWriteBuffer<int> buffer) : IComputeShader
{
    public void Execute()
    {
        buffer[ThreadIds.X] += 10;
    }
}
```
