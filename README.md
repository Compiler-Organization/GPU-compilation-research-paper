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

Each the bytecode of each instruction would be structured like this:
|opcode|operand 1|operand 2|branch|
|--|--|--|--|

branch is a new segment in the instruction bytecode setup. Because we cannot directly jump in threads without diverging branches, we're going to do some witchery in order to trick the GPU to not diverge threads.

[https://co-design.pop-coe.eu/patterns/gpu-branch-divergence.html](https://co-design.pop-coe.eu/patterns/gpu-branch-divergence.html)

# Instructions
## Definitions
* r = register
* qword = quadruple word, 8 bytes
## Table
|opcode|instruction size|mnemonic|Operand 1| Operand 2|
|--|--|--|--|--|
|00|4|Add|r|r|
|01|4|Add|r|qword|
|02|4|Cmp|r|r|
|03|4|Cmp|r|qword|
|04|4|Cmp|qword|qword|
|05|4|C.Branch|r|branch number|

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
