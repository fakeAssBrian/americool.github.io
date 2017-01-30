---
layout: page
title: Contact
permalink: /contact/
feature-img: "img/sample_feature_img.png"
---

<h1 id="form-header"> Hey guys, send me a message! </h1>
<form action="https://getsimpleform.com/messages?form_api_token=97e63a4fa0551a050307c5b249059e18" method="post">
  <!-- the redirect_to is optional, the form will redirect to the referrer on submission -->
  <!-- #2 -->
  <div class="flex">
    <input type='hidden' name='redirect_to' value='http://abeanderson.me/thank-you/' />
    <!-- build local thank you page  -->
    <input type='text' name='name' placeholder='Your Full Name' />
    <input type='email' name='email' placeholder='Your E-mail Address' />
  </div>
  <br><br>
  <textarea name='message' placeholder='Write your message ...'></textarea>
  <br>
  <input type='submit' value='Send Message' />
</form>
