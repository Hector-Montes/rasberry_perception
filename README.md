# rasberry_perception

## Detection


### Detection Backends

Modular detection backends are available in `rasberry_perception` enabling users to utilise deep learning 
frameworks/non-ros methods to detect objects. This functionality is provided via a ros service `GetDetectorResults`. 
This modular approach enables things such as Python3 in Python2 ros environments.

To run the detection backend service run the command below replacing backend with a listed one below or your own:

```bash 
rosrun rasberry_perception detection_server.py backend:="" backend_arg1:=""
```

#### Default `backend:="default"`

The default backend will return an empty object when called.

#### MMDetection `backend:="mmdetection"`

This backend adds support for the popular deep learning framework mmdetection in PyTorch. 
Specifically a private research branch of mmdetection maintained by [Raymond](https://github.com/RaymondKirk).

Before running the backend service, source the appropriate packages from the private repo and pass 
`config_path:=""`, `model_path:=""` and optionally `device=""`.

#### Adding a new detection backend 

Adding custom backends such as TensorFlow, PyTorch, Detectron, Onnx etc. to `rasberry_perception` is easy. 

Just create a custom class as shown below, inherit from the base, and register it's backend name in the detection registry. 
Note you have to place the custom class inside the 
[registry file](src/deep_learning_ros/compatibility_layer/registry.py) found here `src/deep_learning_ros/compatibility_layer/registry.py`.
This can be avoided in future releases if we start to amass a large collection of supported backends.


```python
from rasberry_perception.srv import GetDetectorResultsResponse
from deep_learning_ros.compatibility_layer.detection_server import _DetectorResultsServer
from deep_learning_ros.compatibility_layer.registry import DETECTION_REGISTRY

@DETECTION_REGISTRY.register_detection_backend("CustomBackendName")
class CustomVisionBackend(_DetectorResultsServer):
    def __init__(self, custom_arg1, custom_arg2, default_arg1="hello"):
        # Do your custom initialisation logic here (custom imports etc)
        _DetectorResultsServer.__init__(self)

    def get_detector_results(self, request):
        # Implement your custom backend logic here (return a GetDetectorResultsResponse object)
       return GetDetectorResultsResponse()
```

When launching the detection server via `rosrun` you can pass in arguments to your custom backend as you would usually.
The node will fail if you do not pass any non-default arguments such as `custom_arg1` and `custom_arg2` in the example.

```bash
rosrun rasberry_perception detection_server.py _custom_arg1:="a1" _custom_arg2:="a2" _default_arg1"="world"
```