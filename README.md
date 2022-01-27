# multi-Camera-Split-

Fungsi pada dasarnya berguna menghidupkan 2 kamera dengan menggunakan 1 program kemudian output kamera dipublish dengan menggunakan metode ROS Publish Image.Metode ini memungkinkan apa bila salah satu kamera mengalami disconnect maka program akan tetap mengolah citra pada kamera satunya lagi.
#!/usr/bin/env python
import sys
import cv2
import rospy
from std_msgs.msg import String
from sensor_msgs.msg import Image
from cv_bridge import CvBridge
import numpy
 
def talker():
  pub_1 = rospy.Publisher('raw_image', Image,queue_size=10)
  pub_2 = rospy.Publisher('image_raw', Image,queue_size=10)
  rospy.init_node('omniwheel_camera', anonymous=False)
  rate = rospy.Rate(100) # 20hz
 
  capture_1= cv2.VideoCapture(0)
  capture_1.set(cv2.CAP_PROP_FRAME_WIDTH, 800)
  capture_1.set(cv2.CAP_PROP_FRAME_HEIGHT, 600)
 
  capture_2 = cv2.VideoCapture(2)
  capture_2.set(cv2.CAP_PROP_FRAME_WIDTH, 800)
  capture_2.set(cv2.CAP_PROP_FRAME_HEIGHT, 600)
  br = CvBridge()
  while not rospy.is_shutdown():
       try:
           if(capture_1.isOpened()):
                   [status,img_1] = capture_1.read()
                   left_right_image = numpy.split(img_1, 2, axis=1)
                   image = left_right_image[0]
                   if status == True:
                       rospy.loginfo('publish image_1')
                       pub_1.publish(br.cv2_to_imgmsg(image,"bgr8"))
                       cv2.waitKey(1)
 
           if(capture_2.isOpened()):
                   [status_2,img] = capture_2.read()
                  
                   if status_2 == True:
                       rospy.loginfo('publish image_2')
                       pub_2.publish(br.cv2_to_imgmsg(img,"bgr8"))
                       cv2.waitKey(1)       
       except:
           pass       
if _name_ == '_main_':
   try:
       talker()
   except rospy.ROSInterruptException:
       pass
