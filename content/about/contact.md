---
title: Get in Touch
date: 2026-01-10
weight: 10
---

Whether you have a project in mind, a question about my work, or just want to say hello — drop me a message.

<form id="contact-form" action="https://formspree.io/f/xwvzvjap" method="POST" class="not-prose space-y-3 mt-4">
  <input type="hidden" name="_subject" value="Portfolio Enquiry" />
  <div class="form-control">
    <label class="label pb-1" for="contact-name"><span class="label-text text-xs font-medium">Name</span></label>
    <input id="contact-name" type="text" name="name" required placeholder="Your name" class="input input-bordered input-sm w-full" />
  </div>
  <div class="form-control">
    <label class="label pb-1" for="contact-email"><span class="label-text text-xs font-medium">Email</span></label>
    <input id="contact-email" type="email" name="_replyto" required placeholder="your@email.com" class="input input-bordered input-sm w-full" />
  </div>
  <div class="form-control">
    <label class="label pb-1" for="contact-message"><span class="label-text text-xs font-medium">Message</span></label>
    <textarea id="contact-message" name="message" required placeholder="Your message..." rows="4" class="textarea textarea-bordered textarea-sm w-full resize-none"></textarea>
  </div>
  <button type="submit" id="contact-submit" class="btn btn-primary btn-sm w-full mt-1">Send Message</button>
</form>

<div id="contact-success" style="display:none" class="not-prose alert alert-success mt-4">
  <svg xmlns="http://www.w3.org/2000/svg" class="stroke-current shrink-0 h-6 w-6" fill="none" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" /></svg>
  <span>Thanks! Your message has been sent.</span>
</div>

<script>
document.getElementById('contact-form').addEventListener('submit', async function (e) {
  e.preventDefault();
  var btn = document.getElementById('contact-submit');
  btn.disabled = true;
  btn.textContent = 'Sending…';
  try {
    var res = await fetch(this.action, {
      method: 'POST',
      body: new FormData(this),
      headers: { 'Accept': 'application/json' }
    });
    if (res.ok) {
      this.style.display = 'none';
      document.getElementById('contact-success').style.display = 'flex';
    } else {
      btn.disabled = false;
      btn.textContent = 'Send Message';
    }
  } catch (_) {
    btn.disabled = false;
    btn.textContent = 'Send Message';
  }
});
</script>
