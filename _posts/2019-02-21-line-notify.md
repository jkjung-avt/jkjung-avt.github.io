---
layout: post
comments: true
title: "Sending LINE Notify Messages with Python"
excerpt: "I'd like to share a very useful trick for edge AI devices: sending LINE Notify messages (whenever events are detected).  I also share all source code within the post."
date: 2019-02-21
category: "python"
tags: python
---

I'd like to share a very useful trick I learned recently: sending LINE Notify messages using Python code.

When designing an edge AI device, we often need to have the edge device send some notification to some people whenever an event is detected.  The notification could be an SMS message, an email or else.  I recently came to learn that LINE Notify messages work great in this scenario.  As long as the edge device is connected to the network, it could send such LINE Notify messages, possibly along with some images or video, for free.  I developed Python code to do that, as detailed below.

# Prerequisite

* To test the code below, you'll need to have python3 and the `requests` module installed on the system.

  ```shell
  $ sudo pip3 install requests
  ```

# Reference

* [Using LINE Notify to send messages to LINE from the command-line](https://engineering.linecorp.com/en/blog/using-line-notify-to-send-messages-to-line-from-the-command-line/)

# Code

* The python code is rather straightforward.  So I just attach the full source code below.

  ```python
  """line_notify.py
  
  For sending a LINE Notify message (with or without image)
  
  Reference: https://engineering.linecorp.com/en/blog/using-line-notify-to-send-messages-to-line-from-the-command-line/
  """
  
  import requests
  
  
  URL = 'https://notify-api.line.me/api/notify'
  
  
  def send_message(token, msg, img=None):
      """Send a LINE Notify message (with or without an image)."""
      headers = {'Authorization': 'Bearer ' + token}
      payload = {'message': msg}
      files = {'imageFile': open(img, 'rb')} if img else None
      r = requests.post(URL, headers=headers, params=payload, files=files)
      if files:
          files['imageFile'].close()
      return r.status_code
  
  
  def main():
      import os
      import sys
      import argparse
      try:
          token = os.environ['LINE_TOKEN']
      except KeyError:
          sys.exit('LINE_TOKEN is not defined!')
      parser = argparse.ArgumentParser(
          description='Send a LINE Notify message, possibly with an image.')
      parser.add_argument('--img_file', help='the image file to be sent')
      parser.add_argument('message')
      args = parser.parse_args()
      status_code = send_message(token, args.message, args.img_file)
      print('status_code = {}'.format(status_code))
  
  
  if __name__ == '__main__':
      main()
  ```

# Test

1. Generate a LINE Notify 'token', by following the [Generating personal access tokens](https://engineering.linecorp.com/en/blog/using-line-notify-to-send-messages-to-line-from-the-command-line/) instruction on the referenced web page.  You should copy and save the generated token (a long stream of randomized characters).

   ![LINE Notify token generated](/assets/2019-02-21-line-notify/LINE_TOKEN.png)

2. Set the `LINE_TOKEN` environment variable using the generated token.

   ```shell
   $ export LINE_TOKEN=xxxxxx
   ```

3. Save the above-mentioned source code as `line_nofity.py`.  And execute the following.  (Replace the messages and image file with whatever you'd like to test with.)

   ```shell
   $ python3 line_notify.py --img_file ${HOME}/Pictures/Jensen_Huang.jpg "This is a picture of Jensen Huang taken by myself when I visited NVIDIA last year."
   status_code = 200
   $ python3 line_notify.py "Now I'm sending a pure text message through LINE Notify."
   status_code = 200
   ```

4. Take the above as example, I'd receive the following on my smartphone.

   ![LINE messages on JK's smartphone](/assets/2019-02-21-line-notify/LINE_screenshot.jpg)

# Conclusion

Again, I think LINE Notify messaging is a great way for edge AI devices to send notifications to people, provided the notification is not very frequent (otherwise you might need to pay for the service).  In addition to LINE, this technique should apply similarly to, say, WeChat, Skype, and Facebook Messenger, as well.

With the provided source code, you should be able to do `from line_notify import send_message` and start integrating this functionality into your own Python code.
