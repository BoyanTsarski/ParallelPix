# ParallelPix – Bulk Image Resizer

Final project for the **Concurrent Programming** course.

## Description

ParallelPix reads all images from a folder (including subfolders), resizes them to a target size and
saves them to an output folder with the same structure. The work is split between several parallel
workers, so many images are processed at the same time.

## One job

**Resize one image file** – load it, resize it, save it to the matching path in the output folder.

## Planned features

- **Workers:** configurable number (e.g. 4 or 8). Each worker takes the next job from a shared queue, processes it and takes the next one.
- **Start / Stop / Pause / Resume** buttons that control all workers together.
- **Worker panel:** one row per worker showing its status (Idle / Working / Paused / Finished) and the file it is resizing.
- **Results table:** file name, original size, new size, status (OK / Error), time, worker; plus a progress bar.
- **Responsive UI:** resizing runs in the background, never on the UI thread.

## Shared data

| Data | Protection |
|------|------------|
| Job queue (`Queue<ResizeJob>`) | `lock (_queueLock)` around check + dequeue, so two workers never take the same image. |
| Results list and counters | `lock (_resultsLock)` around every add and read. |

Both locks are objects of the .NET `Lock` class. To avoid deadlocks, locks are never nested, and the slow
work (loading, resizing, saving) is always done outside any lock.
