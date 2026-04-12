The edge preprocessor is the most latency-sensitive piece of your entire pipeline — it runs on the wearable SoC (think Qualcomm Snapdragon, NVIDIA Jetson Orin Nano, or Apple M-series depending on your form factor) and has to keep up with all six streams in real time without draining the battery or overheating. Here's exactly what it does:Each row is clickable to go deeper. Here's what each stage actually does:

**Encode / compress (per modality)**

Video uses H.265 on the hardware codec (never software — you'll fry the CPU and battery). CRF 23 is a good starting point: perceptually lossless for most downstream CV tasks, roughly 60–70% size reduction over raw. Switch to AV1 if your SoC supports it (Snapdragon 8 Gen 2+ does) — 30% smaller than H.265 at equivalent quality.

IMU applies a Kalman filter first (smooths gyro noise, doesn't add latency meaningfully), then delta-encodes the resulting values (each sample stores the diff from the previous), then lz4 compresses the batch. You'll get 5–10× compression over raw floats this way.

Eye gaze depth maps use RVL (Run-Length Variable-length) encoding — a standard for depth frames that exploits the spatial coherence of depth data far better than general-purpose codecs. The 2D gaze point (x, y, confidence) gets quantized to fixed-point integers; sub-pixel precision beyond ~0.1° is noise anyway.

Audio goes through Opus at 16 kbps with 20ms frames, which is the sweet spot for speech and ambient sound. Voice Activity Detection (VAD) runs inline to tag silent frames, saving you from storing hours of silence.

GPS coordinates get quantized to fixed-point with ~10cm precision (7 decimal places of latitude is overkill at 1 Hz), then geohashed for fast spatial indexing later. It's so low-bandwidth that compression barely matters, but consistent encoding does.

Tactile is the most interesting. Full pressure arrays at 1 kHz are huge. Contact event encoding is much smarter: instead of storing the full NxM pressure grid every frame, you store only the cells that changed beyond a threshold (sparse representation) plus run-length encoding of the contact regions. For typical manipulation tasks, this gives 10–20× compression with no meaningful information loss.

**The shared timestamp engine**

This is the most important component in the entire preprocessor. Every stream gets stamped with a monotonic hardware clock value derived from a PTP (Precision Time Protocol) hardware clock, synchronized to a GPS-derived time source when available. The key property you want is that the timestamps are from the same clock domain across all six modalities — not six separate clocks that drift relative to each other. Without this, your time alignment job in the cloud becomes a deconvolution problem instead of a simple join.

In practice on most wearable SoCs, you expose the PTP clock via `CLOCK_TAI` (not `CLOCK_REALTIME`, which jumps on NTP adjustments) and write the timestamp into each encoded packet's header before it hits the local buffer. Store it as a 64-bit nanosecond integer — don't use float, you'll lose precision.

**What the output bus produces**

Every stream exits as a framed, tagged packet with a consistent envelope: `{session_id, device_id, modality, sequence_number, hardware_timestamp_ns, payload_bytes}`. This common envelope is what lets the downstream Kafka producer and the time alignment job treat all six streams uniformly, regardless of how different the raw encodings are.

Want me to sketch out the Protobuf schema for this packet envelope, or go deeper on the timestamp synchronization strategy?