---
layout: page
title: Contact
permalink: /contact/
feature-img: "img/color.png"
---
<!-- FIND BOOTSTRAP CSS PUT IN HEAD, FIND THEME, CHECK BOOKMARK, USE DIVS n'AT & classes - MAKE NOT UGLY ETC. --> 
<!-- #1 -->
<form action="https://getsimpleform.com/messages?form_api_token=97e63a4fa0551a050307c5b249059e18" method="post">
  <!-- the redirect_to is optional, the form will redirect to the referrer on submission -->
  <!-- #2 -->
  <input type='hidden' name='redirect_to' value='full-url/thank-you/' />
  <!-- build local thank you page  -->
  <input type='text' name='name' placeholder='Your Full Name' />
  <input type='email' name='email' placeholder='Your E-mail Address' />
  <textarea name='message' placeholder='Write your message ...'></textarea>
  <input type='submit' value='Send Message' />
</form>
