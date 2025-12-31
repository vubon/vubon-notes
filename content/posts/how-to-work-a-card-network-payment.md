---
title: 'How To Work A Card Network - Payment'
date: 2023-03-02 06:53:33
lastmod: 2023-03-02 06:54:13
draft: false
categories:
  - 'Payments'
aliases:
  - '/how-to-work-a-card-network-payment/'
---

<p id="4b4b" class="pw-post-body-paragraph ig ih hj ii b ij ik il im in io ip iq ir is it iu iv iw ix iy iz ja jb jc jd hc bi" data-selectable-paragraph="">For a long time, I did not write a single article. Suddenly, I thought I should write something about the financial system because I have been working with financial companies for four years already. The concept may be old, but there are still a lot of people who do not know how it works!</p>
<p id="846b" class="pw-post-body-paragraph ig ih hj ii b ij ik il im in io ip iq ir is it iu iv iw ix iy iz ja jb jc jd hc bi" data-selectable-paragraph="">First things first, in this article, we will design a “Card Network” and understand how VISA, Mastercard, JCB cards, and others work. Let’s suppose that “<strong class="ii hk">Vubon</strong>” is the brand name of our card network system.</p>
<p id="a9f5" class="pw-post-body-paragraph ig ih hj ii b ij ik il im in io ip iq ir is it iu iv iw ix iy iz ja jb jc jd hc bi" data-selectable-paragraph="">Let’s break down our system into multiple parts.</p>

<ol class="">
 	<li id="2a87" class="je jf hj ii b ij ik in io ir jg iv jh iz ji jd jj jk jl jm bi" data-selectable-paragraph=""><strong class="ii hk"><em class="jn">Customer (A card holder)</em></strong></li>
 	<li id="9fc3" class="je jf hj ii b ij jo in jp ir jq iv jr iz js jd jj jk jl jm bi" data-selectable-paragraph=""><strong class="ii hk"><em class="jn">Merchant (Service provider, e.g. Food Service, Ride Service, etc)</em></strong></li>
 	<li id="0f9f" class="je jf hj ii b ij jo in jp ir jq iv jr iz js jd jj jk jl jm bi" data-selectable-paragraph=""><strong class="ii hk"><em class="jn">Acquirer Bank (Who provides POS machine, Payment Gateway, etc)</em></strong></li>
 	<li id="0290" class="je jf hj ii b ij jo in jp ir jq iv jr iz js jd jj jk jl jm bi" data-selectable-paragraph=""><strong class="ii hk"><em class="jn">Issuer Bank ( Who provides cards to customers)</em></strong></li>
 	<li id="72eb" class="je jf hj ii b ij jo in jp ir jq iv jr iz js jd jj jk jl jm bi" data-selectable-paragraph=""><strong class="ii hk"><em class="jn">Vubon card Network</em></strong></li>
</ol>
<p id="5793" class="pw-post-body-paragraph ig ih hj ii b ij ik il im in io ip iq ir is it iu iv iw ix iy iz ja jb jc jd hc bi" data-selectable-paragraph=""><strong class="ii hk">Customer: </strong>A customer can obtain a credit card from any issuing bank. However, if the customer wants a debit card, they should have a bank account with the same issuing bank. For instance, let’s say the customer’s name is Roy and he has a credit card with the card number 2222–3333–4444–5555.</p>
<p id="ad5e" class="pw-post-body-paragraph ig ih hj ii b ij ik il im in io ip iq ir is it iu iv iw ix iy iz ja jb jc jd hc bi" data-selectable-paragraph=""><strong class="ii hk">Merchant</strong>: Let’s refer to the merchant as Pathao (which is my ex-company). Pathao is providing food services to customers.</p>
<p id="a269" class="pw-post-body-paragraph ig ih hj ii b ij ik il im in io ip iq ir is it iu iv iw ix iy iz ja jb jc jd hc bi" data-selectable-paragraph=""><strong class="ii hk">Acquirer Bank: </strong>This bank mainly provides services such as a POS machine, payment gateway, etc.</p>
<p id="3b10" class="pw-post-body-paragraph ig ih hj ii b ij ik il im in io ip iq ir is it iu iv iw ix iy iz ja jb jc jd hc bi" data-selectable-paragraph=""><strong class="ii hk">Issuer Bank: </strong>The issuing bank primarily issues credit or debit cards on behalf of the card network company. In this case, Vubon is our card network company, and therefore all card numbers are provided by <strong class="ii hk">Vubon</strong>.</p>
<p id="4de2" class="pw-post-body-paragraph ig ih hj ii b ij ik il im in io ip iq ir is it iu iv iw ix iy iz ja jb jc jd hc bi" data-selectable-paragraph=""><strong class="ii hk">Vubon card network: </strong>As we have already established <strong class="ii hk">Vubon</strong> as a card network company, it will communicate with both the Issuing Bank and the Acquiring Bank.</p>
<p id="384e" class="pw-post-body-paragraph ig ih hj ii b ij ik il im in io ip iq ir is it iu iv iw ix iy iz ja jb jc jd hc bi" data-selectable-paragraph=""><strong class="ii hk">Example:
</strong>Roy, a Pathao customer, orders food and pays using his credit card with the number 2222–3333–4444–5555. What happens behind the scenes? Take a closer look at the diagram for some insights.</p>

<figure>
<img src="/images/Vubon-Card-Network.png" alt="Vubon Card Network diagram" width="710" height="442" />

<p id="46c9" class="pw-post-body-paragraph ig ih hj ii b ij ik il im in io ip iq ir is it iu iv iw ix iy iz ja jb jc jd hc bi" data-selectable-paragraph="">Here’s how the payment process works:</p>

<ul>
	<li><strong class="ii hk">Step 1:</strong> <strong class="ii hk">Roy</strong> submits his card details to Pathao.</li>
	<li><strong class="ii hk">Step 2:</strong> <strong class="ii hk">Pathao</strong> submits the card details to the Acquirer.</li>
	<li><strong class="ii hk">Step 3:</strong> The Acquirer identifies the card brand and submits the card details to <strong class="ii hk">Vubon</strong>.</li>
	<li><strong class="ii hk">Step 4:</strong> <strong class="ii hk">Vubon</strong> identifies the Issuing Bank and requests authorization for the transaction by submitting the transaction details.</li>
	<li><strong class="ii hk">Step 5:</strong> The Issuing Bank checks Roy’s account details, including balance and limitations. If everything is in order, the card network will receive a response from the Issuing Bank. The card network will then relay this response to the Acquirer and the merchant. Finally, Roy receives a successful response for his transaction.</li>
</ul>
<p data-selectable-paragraph="">So, this is a basic explanation of how a card network system works and how a card transaction is processed. In my next article, I will discuss the <strong class="ii hk">3DS system</strong>. It is another mechanism that helps facilitate transaction flows.</p>

</figure>
