---
title: "Face Recognition Using OpenCV & Deep Learning"
excerpt: "Python implementation of face recognition"
header:
  teaser: /assets/images/face_recognition/camera.jpg
  overlay_image: assets/images/face_recognition/camera.jpg
  caption: "Photo credit: [**Unsplash**](https://unsplash.com)"
author_profile: true
categories:
  - Data Science
tags:
  - Deep Learning
  - OpenCV
  - FaceNet
classes: wide
---
## Introduction

This project is focused on building a webcam integrated facial recognition application using pre-trained DNN models and Python's OpenCV library. To recognize faces, the project uses a special kind of neural architecture called "Siamese Network" in which the output vectors (embeddings) of input images are compared against each other to measure the similarity between the two.


The entire pipeline of the project is divided into 4 major parts as mentioned below. The scripts associated with each stage are run in sequence using command prompt to run the facial recognition application.

* Image data generation (imagecapture.py)
* Face detection
* Extraction of image embeddings (extract_embeddings.py)
* Face recognition (recognize_video.py)


## Image Data Generation

For any facial recognition task, the very step is to build user image data so that it can be compared to live facial data in the future for recognition. The python script "imagecapture.py" is used here to capture user images which are then stored on scratch disk for further processing.

Frames from the webcam video are captured and stored on local drive using OpenCV's VideoCapture method as shown in the code below.

##### Sample Code (Refer to "imagecapture.py" script for the entire code)

``` python
#Capture Images
print("Starting Webcam...")
capture = cv2.VideoCapture(0)

image_counter =  1

while True:
    _, frame = capture.read()
    cv2.imshow('imagasde', frame)
    k = cv2.waitKey(100) & 0xff
    if k == 27:
        # ESC pressed
        print("Escape hit. Closing Webcam...")
        break
    elif k == 32:
        # SPACE pressed
        print("writing file")
        image_name = "opencv_frame_{}.png".format(image_counter)
        cv2.imwrite(os.path.join(directory, image_name), frame)
        print("{} written!".format(image_name))
        image_counter += 1

capture.release()
cv2.destroyAllWindows()
```

## Face Detection

Since an image can have multiple objects besides face, it is important for us to crop out just the face part before sending it to the model to extract embeddings. To accomplish this, OpenCV's pre-trained Caffe deep learning model is used. The pre-trained model outputs face detections and associated probabilities along with the coordinates of the detection.

To reduce noise in the detections, a confidence limit is used to filter out weak face detections.

``` python
detector = cv2.dnn.readNetFromCaffe(protoPath, modelPath)

detector.setInput(imageBlob)
detections = detector.forward()

if len(detections) > 0:
		i = np.argmax(detections[0, 0, :, 2])
		confidence = detections[0, 0, i, 2]

		#filter out weak detections
		if confidence > confidence_limit:
			# compute the (x, y)-coordinates of the bounding box for
			# the face
			box = detections[0, 0, i, 3:7] * np.array([w, h, w, h])
			(startX, startY, endX, endY) = box.astype("int")

			# extract the face from the coordinates
			face = image[startY:endY, startX:endX]
			(fH, fW) = face.shape[:2]

			# ensure the face width and height are sufficiently large
			if fW < 20 or fH < 20:
				continue
```

## Extraction of Image Embeddings
The pre-processed image is then inputted to a commonly used pre-trained embedding model called OpenFace - a python and torch implementation of face recognition that is based on the paper "FaceNet: A Unified Embedding for Face Recognition and Clustering".  

The model outputs a 128-D facial embedding vector for every user image in the dataset which is then stored in a pickle dump.

``` python
embedder = cv2.dnn.readNetFromTorch(embedding_model_path)
faceBlob = cv2.dnn.blobFromImage(face, 1.0 / 255,
				(96, 96), (0, 0, 0), swapRB=True, crop=False)
			embedder.setInput(faceBlob)
			vec = embedder.forward()

			# add the name of the person + corresponding face
			# embedding to their respective lists
			knownNames.append(name)
			knownEmbeddings.append(vec.flatten())
			total += 1

# dump the facial embeddings + names to disk
print("[INFO] serializing {} encodings...".format(total))
data = {"embeddings": knownEmbeddings, "names": knownNames}
print(type(data))
f = open(out_embeddings, "wb")
f.write(pickle.dumps(data))
f.close()
```


## Face Recognition

This is the section where the face recognition happens. The image frames from the live video feed are passed through the encoder to get the 128-D embedding vectors. These are then compared to each user embedding stored in the database using the L2 norm distance method. L2 norm distances are calculated for every combination and the user with the minimum L2 norm distance is chosen as the recognized individual.

##### Sample Code (Refer to "recognize_video.py" script for the entire code)
``` python
def who_is_it(vector,database_encode):
	encoding = vector
	min_dist = 100

	for i in range(len(database["embeddings"])):
		db_enc = database["embeddings"][i]
		name = database["names"][i]
		dist = np.linalg.norm(encoding - db_enc)

		if dist < min_dist:
			min_dist = dist
			identity = name
	if not min_dist < 0.55:
		identity = "Not in database"
	print(min_dist)
	return min_dist, identity

faceBlob = cv2.dnn.blobFromImage(face, 1.0 / 255,
				(96, 96), (0, 0, 0), swapRB=True, crop=False)
			embedder.setInput(faceBlob)
			vec = embedder.forward()

			similarity, name = who_is_it(vec, database)
			# perform classification to recognize the face
			# preds = recognizer.predict_proba(vec)[0]
			# j = np.argmax(preds)
			# proba = preds[j]
			# name = le.classes_[j]

			# draw the bounding box of the face along with the
			# associated probability
			text = "{}: {:.2f}".format(name, similarity)
			y = startY - 10 if startY - 10 > 10 else startY + 10
			cv2.rectangle(frame, (startX, startY), (endX, endY),
				(0, 0, 255), 2)
			cv2.putText(frame, text, (startX, y),
				cv2.FONT_HERSHEY_SIMPLEX, 0.45, (0, 0, 255), 2)
```
## Result
<img src="/assets/images/face_recognition/camera.jpg" alt="face recognition" style="width:300px;height:300px;">


For the entire python notebook, please visit *[GitHub](https://github.com/jatinselmokar/opencv-face-recognition-using-facenet-dnn)* link.
