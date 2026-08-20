# Video Walkthrough — MTP Training

A screen-recording walkthrough of the [MTP Training Workflow](./mtp-training-workflow.md).

[![MTP Training Walkthrough](https://img.youtube.com/vi/Cf3GME6iuPg/0.jpg)](https://youtu.be/Cf3GME6iuPg)

Click the thumbnail above to watch on YouTube.

**Notes:**
- A successful `sbatch` submission is not a guarantee the job will work. `sbatch` only validates the `#SBATCH` directive lines (account, time, etc.) at submission time — it doesn't check or run the actual commands inside the script until the job is later dispatched to a compute node. As shown in the video, a script with an unreplaced `<user>` placeholder or a wrong dataset filename will still submit and queue up fine, just like I ran into, and only fail once it actually runs.
- The trained potential is written to `out/18_trained.almtp` inside your working directory, and the job's full console output/log — useful for confirming success or diagnosing errors — is written to `train_<jobid>.out`, also in your working directory, using the job ID `sbatch` printed when you submitted it.
