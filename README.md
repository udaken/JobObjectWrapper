# JobObjectWrapper
A job object allows a group of processes to be managed as a unit. Job objects are namable, securable, sharable objects that control attributes of and assign limits to the processes associated with them. Operations performed on the job object affect all processes associated with the job object.
JobObjectWrapper is a .NET abstraction over the Win32 Job Object. With this library you can create job objects, create and assign a process to the job, control process and job limits, and register for the various process- and job-related notification events.

```csharp
using System;
using JobManagement;
using System.Diagnostics;

namespace ConsoleApplicationTestJob
{
    class Program
    {
        static void Main()
        {
            JobObject jo = new JobObject();
            jo.Events.OnExitProcess += OnExitProcess;
            jo.Limits.ActiveProcessLimit = 4;
            jo.Limits.Affinity = new IntPtr(1);

            ProcessStartInfo si = new ProcessStartInfo("whoami", "/all")
            {
                RedirectStandardOutput = true,
                UseShellExecute = false,
                CreateNoWindow = true
            };

            Process p = jo.CreateProcessSecured(si);
            while (!p.StandardOutput.EndOfStream)
            {
                Console.WriteLine(p.StandardOutput.ReadLine());
            }

            foreach (var process in jo.ConstructAssignedProcessList())
            {
                Console.WriteLine(process.ProcessName + " " + process.Id);
            }
            Console.ReadKey();
            jo.TerminateAllProcesses(42);
        }

        static void OnExitProcess(object sender, ExitProcessEventArgs args)
        {
            Console.WriteLine($"Process {args.TheProcess?.ProcessName ?? args.TheProcessId.ToString()} has exited");
        }
    }
}
```
