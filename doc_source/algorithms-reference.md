# AWS Encryption SDK Algorithms Reference<a name="algorithms-reference"></a>


|  | 
| --- |
|  The information on this page is a reference for building your own encryption library that is compatible with the AWS Encryption SDK\. If you are not building your own compatible encryption library, you likely do not need this information\. To use the AWS Encryption SDK in one of the supported programming languages, see [Programming Languages](programming-languages.md)\.  | 

To build your own library that can read and write ciphertexts that are compatible with the AWS Encryption SDK, you should understand how the SDK implements the supported algorithms to encrypt raw data\. The SDK supports nine algorithm suites\. An implementation specifies the encryption algorithm and mode, encryption key length, key derivation algorithm \(if one applies\), and signature algorithm \(if one applies\)\. The following table contains an overview of each implementation\. By default, the SDK uses the first implementation in the following table\. The list that follows the table provides more information\.


**AWS Encryption SDK Algorithm Suites**  

| Algorithm ID \(in 2\-byte hex\) | Algorithm Name | Data Key Length \(in bits\) | Algorithm Mode | IV Length \(in bytes\) | Authentication Tag Length \(in bytes\) | Key Derivation Algorithm | Signature Algorithm | 
| --- | --- | --- | --- | --- | --- | --- | --- | 
| 03 78 | AES | 256 | GCM | 12 | 16 | HKDF with SHA\-384 | ECDSA with P\-384 and SHA\-384 | 
| 03 46 | AES | 192 | GCM | 12 | 16 | HKDF with SHA\-384 | ECDSA with P\-384 and SHA\-384 | 
| 02 14 | AES | 128 | GCM | 12 | 16 | HKDF with SHA\-256 | ECDSA with P\-256 and SHA\-256 | 
| 01 78 | AES | 256 | GCM | 12 | 16 | HKDF with SHA\-256 | Not applicable | 
| 01 46 | AES | 192 | GCM | 12 | 16 | HKDF with SHA\-256 | Not applicable | 
| 01 14 | AES | 128 | GCM | 12 | 16 | HKDF with SHA\-256 | Not applicable | 
| 00 78 | AES | 256 | GCM | 12 | 16 | Not applicable | Not applicable | 
| 00 46 | AES | 192 | GCM | 12 | 16 | Not applicable | Not applicable | 
| 00 14 | AES | 128 | GCM | 12 | 16 | Not applicable | Not applicable | 

**Algorithm ID**  
A 2\-byte value that uniquely identifies an algorithm's implementation\. This value is stored in the ciphertext's message header\.

**Algorithm Name**  
The encryption algorithm used\. For all algorithm suites, the SDK uses the Advanced Encryption Standard \(AES\) encryption algorithm\.

**Data Key Length**  
The length of the data key\. The SDK supports 256\-bit, 192\-bit, and 128\-bit keys\. The data key is generated by a master key\. For some implementations, this data key is used as input to an HMAC\-based extract\-and\-expand key derivation function \(HKDF\)\. The output of the HKDF is used as the data encryption key in the encryption algorithm\. For more information, see **Key Derivation Algorithm** in this list\.

**Algorithm Mode**  
The mode used with the encryption algorithm\. For all algorithm suites, the SDK uses Galois/Counter Mode \(GCM\)\.

**IV Length**  
The length of the initialization vector \(IV\) used with AES\-GCM\.

**Authentication Tag Length**  
The length of the authentication tag used with AES\-GCM\.

**Key Derivation Algorithm**  
The HMAC\-based extract\-and\-expand key derivation function \(HKDF\) used to derive the data encryption key\. The SDK uses the HKDF defined in [RFC 5869](https://tools.ietf.org/html/rfc5869), with the following specifics:  

+ The hash function used is either SHA\-384 or SHA\-256, as specified by the algorithm ID\.

+ For the extract step:

  + No salt is used\. Per the RFC, this means that the salt is set to a string of zeros\. The string length is equal to the length of the hash function output; that is, 48 bytes for SHA\-384 and 32 bytes for SHA\-256\.

  + The input keying material is the data key received from the master key provider\.

+ For the expand step:

  + The input pseudorandom key is the output from the extract step\.

  + The input info is a concatenation of the algorithm ID followed by the message ID\.

  + The length of the output keying material is the **Data Key Length** described previously\. This output is used as the data encryption key in the encryption algorithm\.

**Signature Algorithm**  
The signature algorithm used to generate a signature over the ciphertext header and body\. The SDK uses the Elliptic Curve Digital Signature Algorithm \(ECDSA\) with the following specifics:  

+ The elliptic curve used is either the P\-384 or P\-256 curve, as specified by the algorithm ID\. These curves are defined in [Digital Signature Standard \(DSS\) \(FIPS PUB 186\-4\)](http://doi.org/10.6028/NIST.FIPS.186-4)\.

+ The hash function used is SHA\-384 \(with the P\-384 curve\) or SHA\-256 \(with the P\-256 curve\)\.