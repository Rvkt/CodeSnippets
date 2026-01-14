An Error Code provide the details/reason for the failure of authentication transaction.  For Error Codes details, Aadhaar number holder may refer to Aadhaar Authentication API document published on UIDAI website.

Below are Error Code List -

“100” – Personal information demographic data did not match.

“200” – Personal address demographic data did not match.

“300” – Biometric data did not match.

“310” – Duplicate fingers used.

“311” – Duplicate Irises used.

“312” – FMR and FIR cannot be used in same transaction.

“313” – Single FIR record contains more than one finger.

“314” – Number of FMR/FIR should not exceed 10.

“315” – Number of IIR should not exceed 2.

“316” – Number of FID should not exceed 1.

“330” – Biometrics locked by Aadhaar holder.

“400” – Invalid OTP value.

“402” – “txn” value did not match with “txn” value used in Request OTP API.

“500” – Invalid encryption of session key.

“501” – Invalid certificate identifier in “ci” attribute of “Skey”.

“502” – Invalid encryption of PID.

“503” – Invalid encryption of Hmac.

“504” – Session key re-initiation required due to expiry or key out of sync.

“505” – Synchronized Key usage not allowed for the AUA.

“510” – Invalid Auth XML format.

“511” – Invalid PID XML format.

“512” – Invalid Aadhaar holder consent in “rc” attribute of “Auth”.

“520” – Invalid “tid” value.

“521” – Invalid “dc” code under Meta tag.

“524” – Invalid “mi” code under Meta tag.

“527” – Invalid “mc” code under Meta tag.

“530” – Invalid authenticator code.

“540” – Invalid Auth XML version.

“541” – Invalid PID XML version.

“542” – AUA not authorized for ASA. This error will be returned if AUA and ASA do not have linking in the portal.

“543” – Sub-AUA not associated with “AUA”. This error will be returned if Sub-AUA specified in “sa” attribute is not added as “Sub-AUA” in portal.

“550” – Invalid “Uses” element attributes.

“551” – Invalid “tid” value.

“553” – Registered devices currently not supported. This feature is being implemented in a phased manner.

“554” – Public devices are not allowed to be used.

“555” – rdsId is invalid and not part of certification registry.

“556” – rdsVer is invalid and not part of certification registry.

“557” – dpId is invalid and not part of certification registry.

“558” – Invalid dih.

“559” – Device Certificate has expired.

“560” – DP Master Certificate has expired.

“561” – Request expired (“Pid->ts” value is older than N hours where N is a configured threshold in authentication server).

“562” – Timestamp value is future time (value specified “Pid->ts” is ahead of authentication server time beyond acceptable threshold).

“563” – Duplicate request (this error occurs when exactly same authentication request was re-sent by AUA).

“564” – HMAC Validation failed.

“565” – AUA license has expired.

“566” – Invalid non-decryptable license key.

“567” – Invalid input (this error occurs when unsupported characters were found in Indian language values, “lname” or “lav”).

“568” – Unsupported Language.

“569” – Digital signature verification failed (means that authentication request XML was modified after it was signed).

“570” – Invalid key info in digital signature (this means that certificate used for signing the authentication request is not valid – it is either expired, or does not belong to the AUA or is not created by a well-known Certification Authority).

“571” – PIN requires reset.

“572” – Invalid biometric position.

“573” – Pi usage not allowed as per license.

“574”– Pa usage not allowed as per license.

“575”– Pfa usage not allowed as per license.

“576” - FMR usage not allowed as per license.

“577” – FIR usage not allowed as per license.

“578” – IIR usage not allowed as per license.

“579” – OTP usage not allowed as per license.

“580” – PIN usage not allowed as per license.

“581” – Fuzzy matching usage not allowed as per license.

“582” – Local language usage not allowed as per license.

“586” – FID usage not allowed as per license. This feature is being implemented in a phased manner.

“587” – Name space not allowed.

“588” – Registered device not allowed as per license.

“590” – Public device not allowed as per license.

“710” – Missing “Pi” data as specified in “Uses”.

“720” – Missing “Pa” data as specified in “Uses”.

“721” – Missing “Pfa” data as specified in “Uses”.

“730” – Missing PIN data as specified in “Uses”.

“740” – Missing OTP data as specified in “Uses”.

“800” – Invalid biometric data.

“810” – Missing biometric data as specified in “Uses”.

“811” – Missing biometric data in CIDR for the given Aadhaar number.

“812” – Aadhaar holder has not done “Best Finger Detection”. Application should initiate BFD to help Aadhaar holder identify their best fingers.

“820” – Missing or empty value for “bt” attribute in “Uses” element.

“821” – Invalid value in the “bt” attribute of “Uses” element.

“822” – Invalid value in the “bs” attribute of “Bio” element within “Pid”.

“901” – No authentication data found in the request (this corresponds to a scenario wherein none of the auth data – Demo, Pv, or Bios – is present).

“902” – Invalid “dob” value in the “Pi” element (this corresponds to a scenarios wherein “dob” attribute is not of the format “YYYY” or “YYYYMM-DD”, or the age is not in valid range).

“910” – Invalid “mv” value in the “Pi” element.

“911” – Invalid “mv” value in the “Pfa” element.

“912” – Invalid “ms” value.

“913” – Both “Pa” and “Pfa” are present in the authentication request (Pa and Pfa are mutually exclusive).

“930 to 939” – Technical error that are internal to authentication server.

“940” – Unauthorized ASA channel.

“941” – Unspecified ASA channel.

“950” – OTP store related technical error.

“951” – Biometric lock related technical error.

“980” – Unsupported option.

“995” – Aadhaar suspended by competent authority.

“996” – Aadhaar cancelled (Aadhaar is no in authenticable status).

“997” – Aadhaar suspended (Aadhaar is not in authenticatable status).

“998” – Invalid Aadhaar Number.

“999” – Unknown error.