+++
title = "Using OS file descriptors to transfer data between processes"
date = 2025-08-09
updated = 2025-08-09
type = "post"
in_search_index = true
[taxonomies]
TIL-Tags = ["OS", "python"]
+++

I was trying to read some data from a python process that I spawned inside Typescript. Initially I was trying to read the data in real time from the python print statements. Whatever I printed in my python script was being read as JSON and being parsed to understand what it is.

example:

```python
# script.py

import json

progress_data = {
    "type": "progress",
    "message": "x out of y files are done"
}
print(json.dumps(progress_data))

values_data = {
    "type": "values",
    "data": [ 1, 2, 3, 4...]
}

print(json.dumps(progress_data))
```

I would call the python script inside the typescript process like this

```typescript

const pythonProcess = spawn(
        executablePath,
        [
          '--threshold',
          similarityThreshold.toString(),
          '--db-path',
          dbPath,
        ],
        {
          cwd: path.dirname(executablePath),
          // fd0=stdin, fd1=stdout, fd2=stderr
          stdio: ['pipe', 'pipe', 'pipe'],
        }
      );
```

One problem I had was I would add some print statements for debugging purpose and leave them there, which would later break my flow in TS side.

But we can just write to a specific file descriptior index and not have different data mix up into one std channel

```python

def log_progress(current: int, total: int, current_file: str | None = None) -> None:
    progress_data = {
        "current": current,
        "total": total,
        "currentFile": current_file,
    }
    try:
        payload = (json.dumps(progress_data) + "\n").encode("utf-8")
        os.write(4, payload) # notice the file descriptor number 4
    except Exception as e:
        print(f"Progress pipe write failed: {e}", file=sys.stderr)

try:
    with os.fdopen(3, 'w') as result_pipe:
        result_pipe.write(json.dumps(result_converted))
        result_pipe.flush()
except Exception as e:
    print(f"Result pipe write failed: {e}", file=sys.stderr)

```

and read them in ts like this

```typescript

const pythonProcess = spawn(
        executablePath,
        [
          '--threshold',
          similarityThreshold.toString(),
          '--db-path',
          dbPath,
        ],
        {
          cwd: path.dirname(executablePath),
          // fd0=stdin, fd1=stdout, fd2=stderr, fd3=result JSON, fd4=progress JSON
          stdio: ['pipe', 'pipe', 'pipe', 'pipe', 'pipe'],
        }
      );

      let errorOutput = '';
      let faceData: any = null;
      // Dedicated buffers
      let progressBuffer = '';
      let resultBuffer = '';

      // Read progress JSON lines from fd 4
      const progressStream = pythonProcess.stdio[4] as NodeJS.ReadableStream | null;
      if (progressStream) {
        progressStream.setEncoding('utf8');
        progressStream.on('data', (chunk: string) => {
          progressBuffer += chunk;
          let lineEnd = progressBuffer.indexOf('\n');
          while (lineEnd !== -1) {
            const line = progressBuffer.substring(0, lineEnd);
            progressBuffer = progressBuffer.substring(lineEnd + 1);
            if (line) {
              try {
                const progressData = JSON.parse(line);
              } catch (e) {
                console.error('Error parsing progress JSON:', e);
              }
            }
            lineEnd = progressBuffer.indexOf('\n');
          }
        });
      }

      // Read result JSON from fd 3
      const resultStream = pythonProcess.stdio[3] as NodeJS.ReadableStream | null;
      if (resultStream) {
        resultStream.setEncoding('utf8');
        resultStream.on('data', (chunk: string) => {
          resultBuffer += chunk;
        });
        resultStream.on('end', () => {
          if (resultBuffer) {
            try {
              faceData = JSON.parse(resultBuffer);

            } catch (e) {
              console.error('Failed to parse result JSON:', e);
            }
          }
        });
      }
```


# What is fd 3 and why four entries in stdio?
In Unix-like processes, file descriptors map as:
1. fd 0 = stdin
2. fd 1 = stdout
3. fd 2 = stderr
4. fd 3, 4, … = extra descriptors you explicitly provide

In Node’s spawn, stdio lets you define what the child gets:
['pipe', 'pipe', 'pipe'] gives the child stdin/stdout/stderr as pipes you can read/write via child.stdio[0..2].
Adding more entries like ['pipe', 'pipe', 'pipe', 'pipe'] creates an extra pipe, exposed to the child as fd 3, and to the parent as child.stdio[3].
In Python, you can open and write to that extra pipe with os.fdopen(3, 'w'). That’s how we deliver the final JSON without touching stdout.