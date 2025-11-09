# First observation

<img width="959" height="898" alt="image" src="https://github.com/user-attachments/assets/b0c5b134-951d-4543-a07d-4665697330ab" />


* Both tasks are pinned to the same core: `pro_cpu = 0`. That is why the log only shows Core 0. Despite the title, there is no actual dual-core execution.
* Priorities: `Task H` has priority 2 and `Task L` has priority 1. In FreeRTOS, higher number means higher priority. Since neither task ever blocks, `Task H` preempts almost all the time. The screenshot matches this: one print from Task L, then repeated prints from Task H.
* `hog_delay()` is a busy wait. It wastes CPU cycles, timing is not accurate, and it prevents the task from yielding.
* The banner says "Priority Inheritance Demo", but the code does not demonstrate priority inheritance. There is no mutex or shared resource that could cause priority inversion.
* Comment mismatch: `Task H` is labeled "low priority" in the comment, but its numeric priority is higher than `Task L`.
* `app_cpu` is defined but never used. `setup` deletes itself with `vTaskDelete(NULL)`, so `loop()` is never reached, which is correct under FreeRTOS on ESP32.

If the goal is to show two cores and fairer scheduling:

```cpp
// Pin each task to a different core
xTaskCreatePinnedToCore(doTaskL, "Task L", 2048, NULL, 1, NULL, pro_cpu);
xTaskCreatePinnedToCore(doTaskH, "Task H", 2048, NULL, 2, NULL, app_cpu);

// Replace busy wait with a real delay inside each task
vTaskDelay(time_hog / portTICK_PERIOD_MS);  // yields the CPU
```

Optionally add `vTaskDelay(1)` or `taskYIELD()` inside long loops to prevent starvation when experimenting with busy waits.

# Second observation

<img width="955" height="896" alt="{2F2D7216-539A-4EA0-A214-819468DE8CF4}" src="https://github.com/user-attachments/assets/9f766183-d8f1-4718-a35b-78202800d0f1" />


* Output shows Task L on Core 0 and Task H on Core 1. CPU affinity is working.
* The 99% load matches the busy wait in `hog_delay`, which keeps both cores spinning.
* Message order is not strictly alternating. Seeing Task H twice in a row is normal due to higher priority plus Serial buffering.
* The 200 ms delay is approximate. The nested loops depend on clock speed, so timing on the S3 will drift.
* If you want lower load and more regular logs, replace `hog_delay(time_hog)` with `vTaskDelay(pdMS_TO_TICKS(time_hog))`.
* The banner says Priority Inheritance Demo, but the sketch is demonstrating core pinning, not priority inheritance.
* On ESP32 S3 the core IDs are still 0 and 1. Using `pro_cpu` and `app_cpu` as aliases is fine even if the names come from the original ESP32.

