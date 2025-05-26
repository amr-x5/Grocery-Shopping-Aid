# Grocery-Shopping-Aid
ROS-Based Robotic Camera System for Visually Impaired Users

## Date Created: Jun 2024

This project implements a ROS-based robotic camera system designed to assist visually impaired users. It leverages YOLOv10 for object detection and Automatic Speech Recognition (ASR) libraries to identify and describe items, such as grocery products. The system then communicates this information to the user via text-to-speech, achieving a 96% detection accuracy in describing items.


## 🛠️ Functionality

The system performs the following key functions:

1.  **Audio Input:** Captures audio commands or questions from the user using a microphone.
2.  **Speech Recognition:** Converts the captured audio into text using an ASR library.
3.  **Object Detection:** Utilizes a camera and the YOLOv10 model (via the `darknet_ros` package) to detect objects in the camera's field of view.
4.  **Information Processing:** Combines the recognized speech with the detected object's class to retrieve relevant information (e.g., price, location, description) about the object.
5.  **Text-to-Speech Output:** Converts the processed textual information back into audible speech, providing a description or answer to the user.

---

## 📦 ROS Packages

The Catkin workspace (`catkin_ws`) for this project is organized into the following main ROS packages:

* **`audio_recognition`**: Handles capturing audio and performing speech-to-text conversion.
    * **Node**: `audio_recognition_node.py` (or `audio_processor`)
    * **Publishes**: Recognized speech text to `/speech_recognition/audio` (std_msgs/String).
* **`robot_package`**: The core package that processes information from object detection and speech recognition.
    * **Node**: `object_detection_processor.py`
    * **Subscribes**:
        * `/darknet_ros/bounding_boxes` (darknet_ros_msgs/BoundingBoxes) for object detection data.
        * `/speech_recognition/audio` (std_msgs/String) for recognized speech.
    * **Publishes**: Text to be spoken to `tts_text` (std_msgs/String).
    * **Functionality**: Contains a dictionary of known objects with their properties (price, location, description). When an object is detected and a relevant question is asked (e.g., "what is this?", "where is the...?", "how much is..."), it formulates an appropriate response.
* **`tts_package`**: Responsible for converting text messages into speech.
    * **Node**: `text_to_speech_node.py`
    * **Subscribes**: `tts_text` (std_msgs/String) for text messages.
    * **Functionality**: Uses the `sound_play` library to vocalize the received text.
* **`darknet_ros`** (Assumed external or submodule): Provides the YOLO-based object detection capabilities. It publishes bounding boxes of detected objects and their classes.
    * **Publishes**: `/darknet_ros/bounding_boxes` (darknet_ros_msgs/BoundingBoxes), `/darknet_ros/object_count` (darknet_ros_msgs/ObjectCount), etc.

---

## 🔌 Key ROS Topics

* `/darknet_ros/bounding_boxes` (`darknet_ros_msgs/BoundingBoxes`): Carries information about detected objects, including their class and bounding box coordinates.
* `/speech_recognition/audio` (`std_msgs/String`): Transports the text recognized from the user's speech.
* `tts_text` (`std_msgs/String`): Carries the textual response that the robot needs to speak.

---

## ⚙️ System Dependencies

### ROS & Catkin:
* ROS (Robot Operating System) - Melodic Morenia is used in this project.
* Catkin (Build System)

### ROS Packages:
* **`rospy`**: Python client library for ROS.
* **`std_msgs`**: Standard ROS messages, including `String`.
* **`sound_play`**: For text-to-speech capabilities.
* **`darknet_ros_msgs`**: Custom messages for object detection results from `darknet_ros`.
* **`cv_bridge`**: To convert between ROS Image messages and OpenCV images.
* **`image_transport`**: For publishing and subscribing to images.
* **`nodelet`**: For running multiple algorithms in a single process.
* **`actionlib`**: For creating and interacting with ROS actions.
* **`message_filters`**: For synchronizing messages from different topics.

### External Libraries:
* **YOLOv10** (via `darknet_ros`): For real-time object detection.
* **ASR (Automatic Speech Recognition) Libraries**:
    * `speech_recognition` (Python library): Used in `audio_recognition_node.py`.
    * `sounddevice` (Python library): For audio capture.
    * `soundfile` (Python library): For reading/writing audio files (though primarily used for recording to bytes in this context).
* **OpenCV**: Used by `darknet_ros` and potentially `cv_bridge`.
* **Python 2.7**

---

## 🚀 Setup and Installation

1.  **Install ROS Melodic**: Follow the official ROS Melodic installation guide.
2.  **Create a Catkin Workspace**:
    ```bash
    mkdir -p ~/catkin_ws/src
    cd ~/catkin_ws/
    catkin_make
    ```
3.  **Source the Workspace**:
    ```bash
    source devel/setup.bash
    ```
4.  **Clone the Repository**:
    ```bash
    cd ~/catkin_ws/src
    # git clone <repository_url> . # Assuming this README will be in the root of the project
    ```
    Ensure all packages (`audio_recognition`, `robot_package`, `tts_package`, and `darknet_ros` along with its messages `darknet_ros_msgs`) are correctly placed within the `src` directory. If `darknet_ros` is a submodule, initialize and update it.
5.  **Install Dependencies**:
    * Install system dependencies for ROS packages (e.g., `sound_play`, `OpenCV`, etc.) if not already present.
        ```bash
        sudo apt-get install ros-melodic-sound-play ros-melodic-cv-bridge ...
        ```
    * Install Python dependencies for ASR:
        ```bash
        pip install SpeechRecognition sounddevice soundfile numpy pyaudio # Pyaudio might be needed by sounddevice or SpeechRecognition
        ```
    * Ensure `darknet_ros` dependencies are met (this usually involves compiling Darknet). Refer to the `darknet_ros` package's own installation instructions.
6.  **Build the Workspace**:
    ```bash
    cd ~/catkin_ws/
    catkin_make
    ```
    Address any compilation errors by installing missing dependencies.

---

## ▶️ Usage / Running the System

1.  **Source the Workspace**:
    ```bash
    source ~/catkin_ws/devel/setup.bash
    ```
2.  **Launch the System**:
    You will typically use `roslaunch` to start all the necessary nodes. A master launch file would be ideal, but individual nodes can also be run using `rosrun`.

    * **Run `darknet_ros`**: This usually has its own launch file. For example:
        ```bash
        roslaunch darknet_ros yolo_v3.launch # Or a YOLOv10 specific launch file
        ```
    * **Run Audio Recognition Node**:
        ```bash
        rosrun audio_recognition audio_recognition_node.py
        ```
    * **Run Robot Package Node (Object Detection Processor)**:
        ```bash
        rosrun robot_package object_detection_processor.py
        ```
    * **Run TTS Package Node**:
        ```bash
        rosrun tts_package text_to_speech_node.py
        ```
3.  **Interact**: Once all nodes are running, the system should be ready to:
    * Listen for your voice commands/questions via the microphone.
    * Detect objects in front of the camera.
    * Verbally respond with information about the detected objects based on your questions.

