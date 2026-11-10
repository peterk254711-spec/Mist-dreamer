// Backend: Initiate Payment
const { Daraja } = require("@lumierelabs/daraja");
const daraja = Daraja({
  consumerKey: process.env.DARAJA_KEY,
  consumerSecret: process.env.DARAJA_SECRET,
  shortcode: process.env.DARAJA_SHORTCODE, // Your Paybill/Till
  passkey: process.env.DARAJA_PASSKEY,
  callbackUrl: "https://yourdomain.com/api/mpesa/callback",
  environment: "production" // or "sandbox"
});

app.post("/api/pay", async (req, res) => {
  const { phone } = req.body; // User's Safaricom number
  try {
    const push = await daraja.stkPush({
      transactionType: "CustomerPayBillOnline", // or CustomerBuyGoodsOnline
      amount: 150,
      partyA: phone,
      phoneNumber: phone,
      accountReference: "MIST_DREAMER_ACCESS",
      transactionDesc: "One-time chat access"
    });
    // Save push.CheckoutRequestID to DB linked to this user
    res.json({ checkoutRequestId: push.CheckoutRequestID });
  } catch (err) { res.status(500).send(err.message); }
});

// Backend: Callback Listener (MUST BE PUBLICLY ACCESSIBLE HTTPS URL)
app.post("/api/mpesa/callback", (req, res) => {
  const { Body: { stkCallback } } = req.body;
  const { ResultCode, CheckoutRequestID } = stkCallback;
  if (ResultCode === 0) {
    // Payment Success: Update User DB status to 'paid'
    // Use the CheckoutRequestID to find the user and unlock chat
  }
  res.json({ Result: "OK" }); // Acknowledge receipt to Safaricom
});
