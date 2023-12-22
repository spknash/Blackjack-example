# Blackjack-example

from flask import Flask, request, jsonify
import OpenSSL.crypto
import datetime

app = Flask(__name__)

@app.route('/est/simpleenroll', methods=['POST'])
def simple_enroll():
    # Retrieve the CSR from the request
    csr_data = request.data

    # Load the CSR
    try:
        csr = OpenSSL.crypto.load_certificate_request(OpenSSL.crypto.FILETYPE_PEM, csr_data)
    except OpenSSL.crypto.Error:
        return jsonify({"error": "Invalid CSR format"}), 400

    # In a real-world scenario, you should authenticate the client here

    # Generate the certificate (this is a simplified example)
    # In production, you'd use your CA to sign the certificate
    cert = OpenSSL.crypto.X509()
    cert.set_serial_number(1000)  # In real-world, this should be a unique serial number
    cert.gmtime_adj_notBefore(0)
    cert.gmtime_adj_notAfter(31536000)  # Valid for one year
    cert.set_issuer(csr.get_subject())  # In a real CA, this should be your CA's subject
    cert.set_subject(csr.get_subject())
    cert.set_pubkey(csr.get_pubkey())

    # Sign the certificate with your CA's private key (example uses CSR's key)
    private_key = OpenSSL.crypto.PKey()
    private_key.generate_key(OpenSSL.crypto.TYPE_RSA, 2048)
    cert.sign(private_key, 'sha256')

    cert_pem = OpenSSL.crypto.dump_certificate(OpenSSL.crypto.FILETYPE_PEM, cert)

    return cert_pem, 200

if __name__ == '__main__':
    app.run(debug=True)
