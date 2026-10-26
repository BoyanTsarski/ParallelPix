# ParallelPix – Bulk Image Resizer

Final exam project for the **Concurrent Programming** course.

## Description

ParallelPix is a desktop application that takes a folder full of images, finds every image inside it
(including all subfolders) and resizes them to a chosen target size. The work is split between several
parallel workers, so hundreds of images are processed at the same time instead of one by one.
The resized images are written to an output folder that keeps exactly the same folder structure as the input.

## What is one job?

**One job = resize one image file.**

A job contains:
- the full path of the source image
- the path where the resized image must be saved (same relative path inside the output folder)
- the target size (width × height, or max side with kept aspect ratio)

When the user clicks **Start**, the app scans the input folder recursively and creates one job for every
`.jpg`, `.jpeg`, `.png`, `.bmp` and `.gif` file it finds.

## Planned features

### Workers
- A configurable number of workers (e.g. 2, 4, 8) chosen in the UI before starting.
- Every worker runs in its own thread / task and repeats the same loop:
  take the next job from the shared queue → load the image → resize it → save it → add the result → take the next job.
- When the queue is empty, the worker finishes.

### Controls
| Button     | What it does |
|------------|--------------|
| **Start**  | Scans the input folder, fills the job queue and starts all workers. |
| **Stop**   | Cancels all workers. Jobs that were not processed stay unprocessed. |
| **Pause**  | All workers stop after their current image and wait. |
| **Resume** | All paused workers continue taking jobs from the queue. |

### Showing the workers
A list/panel with one row per worker that shows live:
- worker number (Worker #1, Worker #2, …)
- status: *Idle / Working / Paused / Finished*
- the file it is resizing right now

### Showing the results
- A table of processed images with columns: **File name**, **Original size**, **New size**, **Status** (OK / Error), **Time (ms)**, **Worker**.
- A progress bar and counters: processed / total / errors.

### Responsive UI
The UI thread never does the resizing itself. Workers run in the background and only send
updates to the UI, so the window can always be moved, scrolled and clicked.

## Shared data and how it is protected

| Shared data | Used by | Protection |
|-------------|---------|------------|
| **Job queue** (`Queue<ResizeJob>`) | All workers take jobs from it | Every `Dequeue` is done inside `lock (_queueLock)`, so two workers can never get the same image. |
| **Results list** (`List<ResizeResult>`) | All workers add results; the UI reads them | Every `Add` and every read/copy is done inside `lock (_resultsLock)`. |
| **Counters** (processed, errors) | All workers increase them | Updated inside the results lock (or with `Interlocked.Increment`). |
| **Worker status array** | Each worker writes its own status, the UI reads all | Each worker only writes to its own slot; the UI reads under a lock. |

### Possible problems and how I avoid them
- **Race condition:** without the lock, two workers could check `queue.Count > 0` at the same time and both
  try to take the last job, or two `List.Add` calls could overwrite each other. → Check and dequeue happen
  together inside one lock.
- **Deadlock:** could happen if one worker holds the queue lock and waits for the results lock while another
  does the opposite. → Locks are never nested: a worker releases the queue lock **before** it starts resizing
  and only then takes the results lock. The slow work (loading/resizing/saving the image) is always done
  **outside** any lock.
- **Output folders:** two workers may need to create the same subfolder at once. →
  `Directory.CreateDirectory` is safe to call when the folder already exists.

## Technologies
- C# / .NET
- Windows Forms or WPF (UI)
- `Thread` / `Task`, `lock` with the `Lock` class
- Image processing library (e.g. `System.Drawing` or ImageSharp)

## Status
🚧 Kickoff – planning and repository setup. No working code yet.
