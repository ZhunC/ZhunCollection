---
share: "True"
tags:
  - Edge_ML
  - -embedded
postType: post
categories: Note
---


TensorFlow Lite provides a minimal runtime for executing trained TensorFlow models on embedded systems without dynamic memory allocation or OS dependencies. The workstream consists of setup, model loading, operator resolution, interpreter initialization, tensor allocation, and inference.

## Example Setup and Inference Workflow

```cpp
void setup() {
  // Set up logging
  static tflite::MicroErrorReporter micro_error_reporter;
  error_reporter = &micro_error_reporter;

  // Map the model into a usable data structure
  model = tflite::GetModel(g_sine_model_data);
  if (model->version() != TFLITE_SCHEMA_VERSION) {
    error_reporter->Report(
      "Model provided is schema version %d not equal "
      "to supported version %d.",
      model->version(), TFLITE_SCHEMA_VERSION);
    return;
  }

  // Register all operations used by the model
  static tflite::ops::micro::AllOpsResolver resolver;

  // Build an interpreter to run the model
  static tflite::MicroInterpreter static_interpreter(
    model, resolver, tensor_arena, kTensorArenaSize, error_reporter);
  interpreter = &static_interpreter;

  // Allocate memory from the tensor arena
  TfLiteStatus allocate_status = interpreter->AllocateTensors();
  if (allocate_status != kTfLiteOk) {
    error_reporter->Report("AllocateTensors() failed");
    return;
  }
}
````

### Explanation

1. `MicroErrorReporter` initializes logging for all components.
    
2. `GetModel()` maps the compiled `.tflite` model into memory without copying.
    
3. Schema version is checked to ensure compatibility.
    
4. `AllOpsResolver` loads all kernel implementations needed for inference.
    
5. `MicroInterpreter` binds the model, resolver, and tensor memory.
    
6. `AllocateTensors()` assigns memory slices for model tensors within a static arena.
    

The tensor arena is a fixed-size byte buffer defined as:

```cpp
constexpr int kTensorArenaSize = 2 * 1024;
uint8_t tensor_arena[kTensorArenaSize];
```

No dynamic allocation is performed. All buffers are preallocated in this arena.

---

## Inference Example

```cpp
// Get handles to input and output tensors
TfLiteTensor* input  = interpreter->input(0);
TfLiteTensor* output = interpreter->output(0);

// Place the calculated x value in the model's input tensor
input->data.f[0] = x_val;

// Run inference and report any errors
TfLiteStatus invoke_status = interpreter->Invoke();
if (invoke_status != kTfLiteOk) {
  error_reporter->Report("Invoke failed on x_val: %f\n",
                         static_cast<double>(x_val));
  return;
}

// Read the predicted y value from the model's output tensor
float y_val = output->data.f[0];
```

### Explanation

1. `input(0)` and `output(0)` return pointers to the model’s first input and output tensors.
    
2. The model input is written directly into `input->data.f`.
    
3. `interpreter->Invoke()` executes the neural network inference.
    
4. The result is read from `output->data.f`.
    

### Example Loop

```cpp
while (true) {
  float x_val = ReadSensor();
  input->data.f[0] = x_val;
  interpreter->Invoke();
  float y_val = output->data.f[0];
  Actuate(y_val);
}
```

This loop demonstrates continuous real-time inference using sensor data as input and actuator control as output.

---

## Summary Table

|Stage|Function / Class|Role|
|---|---|---|
|Model Loading|`GetModel()`|Maps `.tflite` file into memory|
|Error Reporting|`MicroErrorReporter`|Handles runtime messages|
|Operator Resolution|`AllOpsResolver`|Registers operator implementations|
|Interpreter|`MicroInterpreter`|Core runtime for inference|
|Tensor Allocation|`AllocateTensors()`|Assigns static memory for tensors|
|Inference|`Invoke()`|Executes model graph|
|Input/Output|`input()`, `output()`|Accesses model I/O buffers|

---

## Key Principles

- All memory is statically allocated; no `malloc` or dynamic allocation.
    
- Execution is deterministic with predictable timing and memory usage.
    
- Platform-independent and runs on bare-metal MCUs, DSPs, and accelerators.
    
- Integrates directly into firmware with minimal dependencies.
    

---
## Source

Repo: [tensorflow/tensorflow/lite/experimental/micro/examples/micro_speech at be4f6874533d78f662d9777b66abe3cdde98f901 · tensorflow/tensorflow · GitHub](https://github.com/tensorflow/tensorflow/tree/be4f6874533d78f662d9777b66abe3cdde98f901/tensorflow/lite/experimental/micro/examples/micro_speech)
Book: TinyML _ Machine Learning with TensorFlow Lite on Arduino, Pete Warden, Daniel Situnayake, 2019