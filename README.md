# First observation

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
