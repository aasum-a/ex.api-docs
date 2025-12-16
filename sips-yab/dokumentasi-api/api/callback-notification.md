# API Payment

Endpoint ini bertindak sebagai _Listener_ untuk menerima notifikasi status bayar dari Payment Gateway.

> **⚠️ SECURITY WARNING**
> Endpoint ini **rawan serangan manipulasi** jika tidak memvalidasi Signature/Secret Key dari Payment Gateway. Pastikan logic mengecek header otentikasi.

## Callback Notification

- **URL**: `POST /api/payment/callback-notifications`
- **Controller**: `Main\Payment\PaymentController@callback_notifications`
- **Access**: Public (Webhook)
- **Body Payload**:
  - `type`: "settlement", "expire", etc.
  - `transaction_id`: ID dari Payment Gateway.
  - `order_id`: ID Order sistem lokal.
  - `data`: Array detail transaksi.
- **Desc**: Menerima status pembayaran. Jika status sukses, sistem akan mengupdate tagihan siswa menjadi LUNAS.
