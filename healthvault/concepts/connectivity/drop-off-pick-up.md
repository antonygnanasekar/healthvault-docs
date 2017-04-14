Drop-off pick-up (DOPU)
=======================

Drop-off and pick-up (DOPU) is a way for an application to send health data to a user without requiring a patient-facing user interface. It's a good choice for applications that don't have a user interface and want a one-time push of data to users. For an ongoing relationship with users, see [Patient Connect](/healthvault/concepts/connectivity/patient-connect.md).

For example, a lab may support sending results to HealthVault, but it doesn't have a website that allows users to log in and connect via the online web user approach. It's also not interested in establishing a connection with the user and exchanging data on an ongoing basis. In this case it can create a DOPU package containing the lab results and send it to HealthVault for the user to pick up.

Making the connection
---------------------

A typical DOPU connection is made in the following way:

1.  A patient visits a lab and wants to store her lab results in her HealthVault account.

2.  The lab's application creates a DOPU package in HealthVault containing the patient's results, the patient's ID in the lab's system, a friendly name for the patient, and a secret question. The secret answer is used to encrypt the DOPU package and to later verify the identity of the user.

3.  HealthVault returns an identity code for the DOPU package.

4.  The application then sends the user an email containing the identity code and a link to HealthVault. (Alternatively, the lab could give the patient a print-out with the information.)

5.  Later, the patient goes to the URL provided by the lab and enters the identity code. She will also create an account if she doesn't already have one.

6.  The patient is prompted for the answer to her secret question, and when she enters it correctly, this validates her identity.

7.  The patient then selects the HealthVault record in which the lab results should be stored.

8.  The data from the DOPU package is written to the patient's record.

9.  The patient receives confirmation, which includes the Patient Connect Success message that was configured in the Application Configuration Center (see "Application Configuration" below).

Package properties
------------------

<table>
<colgroup>
<col width="50%" />
<col width="50%" />
</colgroup>
<thead>
<tr class="header">
<th><p>Property</p></th>
<th><p>Description</p></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p>Identity code</p></td>
<td><p>The unique code generated by HealthVault that identifies the package.</p>
<p>Users begin the process of picking up connect packages by going to <span class="literalValue">https://  <span class="parameter">shellhostname</span>/redirect.aspx?target=PICKUP</span> and entering their identity code manually into the website. You can also provide the identity code as a query string parameter so users don't have to manually enter it themselves. For example:  <span class="literalValue">https:// <span class="parameter">shellhostname</span>/redirect.aspx?target=PICKUP&amp;targetqs=packageid%3d  <span class="parameter">JKYZ-QNMN-VHRX-ZGNR-GZNH</span> </span>. See Shell redirect interface for more details on using the <a href="shell-redirect-interface.md" id="PageContent_14092_3">Shell Redirect Interface</a>.</p></td>
</tr>
<tr class="even">
<td><p>Friendly name</p></td>
<td><p>A friendly name that can be presented to the user after the user successfully answers the challenge question. The friendly name should be something that is recognizable and distinguishes one package from another so that the user may choose the expected record for receiving the data.</p>
<p>For example, a mother of two children may want to name her packages after each child so she can distinguish one child's package from the other's.</p></td>
</tr>
<tr class="odd">
<td><p>Secret question</p></td>
<td><p>A challenge question posed to the user once the identity code has been successfully entered. We recommend that the question be personal and easy to answer in one word. An empty question is not allowed.</p></td>
</tr>
<tr class="even">
<td><p>Secret answer</p></td>
<td><p>The secret answer provided by the user. This should be used to encrypt the package data, so when the user picks up the package, she will have to enter the answer correctly in order to decrypt the package data and verify her identity. The secret answer is not stored in HealthVault.</p></td>
</tr>
<tr class="odd">
<td><p>External ID</p></td>
<td><p>A unique identifier supplied by the application for identifying the package. This value is used to map the patient's information in the application to the package. For example, this could be the patient identifier used to store information in the application's database.</p></td>
</tr>
<tr class="even">
<td><p>Package data</p></td>
<td><p>A Password Protected Package thing containing the data to send to the user. Its data-other must be Base64 encoded and must contain the encrypted data which HealthVault will be able to decrypt with the user's answer to the secret question. The decoded and decrypted data must be a sequence of thing elements as defined by the following schema:</p>
<code class="source-code">&lt;sequence&gt;    &lt;element         name=&quot;thing&quot;         type=&quot;wc-thing:Thing&quot;         minOccurs=&quot;1&quot;         maxOccurs=&quot;unbounded&quot; /&gt;&lt;/sequence&gt; </code></td>
</tr>
</tbody>
</table>

Platform XML methods
--------------------

<table>
<colgroup>
<col width="50%" />
<col width="50%" />
</colgroup>
<thead>
<tr class="header">
<th><p>Method</p></th>
<th><p>Description</p></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p>CreateConnectPackage</p></td>
<td><p>Creates a DOPU package.</p></td>
</tr>
<tr class="even">
<td><p>AllocatePackageId</p></td>
<td><p>Allocates an identity code for a DOPU package. Since sending the package can take a long time, applications can call this method to get an identity code to give to the patient immediately and then later upload the package data by calling CreateConnectPackage with the identity code returned by this method.</p></td>
</tr>
<tr class="odd">
<td><p>UpdateExternalId</p></td>
<td><p>Updates the external ID for a DOPU package.</p></td>
</tr>
<tr class="even">
<td><p>BeginPutConnectPackageBlob</p></td>
<td><p>Initiates uploading a blob associated with the DOPU package via the Streaming API.</p></td>
</tr>
<tr class="odd">
<td><p>DeletePendingConnectPackage</p></td>
<td><p>Deletes a pending DOPU package.</p></td>
</tr>
</tbody>
</table>

.NET SDK
--------

The HealthVault .NET SDK contains classes and methods in the [Microsoft.Health.Package](sdks/dotnet/microsoft.health.package.yml) namespace for creating and managing DOPU packages.

Creating the package
--------------------

A DOPU package can be created in two ways:

1.  Call CreateConnectPackage with the package data.

2.  Call AllocatePackageId to get an identity code, and then call CreateConnectPackage with the package data plus the identity code.

The latter approach allows applications to give identity codes to users before actually uploading data to HealthVault. This is useful for large packages that may take a long time to upload.

Multiple packages are allowed for a given external ID, but a package cannot be updated once created.

&gt;
Package data
------------

The data in a DOPU package is an encrypted list of HealthVault items. It is encapsulated in the [Password Protected Package](http://developer.healthvault.com/pages/types/type.aspx?id=c9287326-bb43-4194-858c-8b60768f000f) type, which consists of metadata about the encryption algorithm and an inline blob that is the encrypted data itself.

The unencrypted data should be a sequence of thing XMLs of any HealthVault data type. Items can also contain inline or streamed blobs, but items with streamed blobs cannot be <a href="digital-signatures.md" id="PageContent_14092_5">digitally signed</a> inside a DOPU package.

For example:

```xml
<thing>  
    <type-id>3d34d87e-7fc1-4153-800f-f56592cb0d17</type-id>  
    <data-xml>    
        <weight>     
            <when>        
                <date>          
                    <y>2006</y>         
                    <m>12</m>          
                    <d>22</d>        
                </date>      
            </when>      
            <value>        
                <kg>125</kg>        
                <display units="lb" units-code="lb">275.5625</display>      
            </value>    
        </weight>  
    </data-xml>
</thing>
<thing>  
    <type-id>3d34d87e-7fc1-4153-800f-f56592cb0d17</type-id>  
    <data-xml>    
        <weight>      
            <when>        
                <date>          
                    <y>2006</y>          
                    <m>12</m>          
                    <d>22</d>        
                </date>      
            </when>      
            <value>        
                <kg>114</kg>        
                <display units="lb" units-code="lb">251.313</display>
            </value>    
        </weight>  
    </data-xml>
</thing> 
```
Adding blobs to package data items is very similar to adding blobs to regular items – the only difference is that you need to call BeginPutConnectPackageBlob instead of BeginPutBlob when initiating a streamed blob for DOPU packages. Using the blob URL returned by BeginPutConnectPackageBlob, you stream the blob data to HealthVault and add the blob metadata to the thing XML as with regular items.

To preserve the confidentiality of the DOPU package, the data should be encrypted based on the user's secret answer before uploading to HealthVault. The encryption key should be derived using the user's secret answer and a salt and iteration count specified by the app. (See the IETF RFC 2892 as a reference.) To ensure case-insensitivity, HealthVault converts the secret answer to lowercase when attempting to decrypt the package, so the answer should be converted to lowercase when encrypting the package as well. The .NET SDK does this automatically. In addition, the unencrypted data should include a Hash-based Message Authentication Code (HMAC) based on the same salt as the encryption key in order for HealthVault to confirm the integrity of the data when the package is picked up.

Supported encryption algorithms:

<table>
<colgroup>
<col width="25%" />
<col width="25%" />
<col width="25%" />
<col width="25%" />
</colgroup>
<thead>
<tr class="header">
<th><p>Algorithm</p></th>
<th><p>Block size (bits)</p></th>
<th><p>Key size (bits))</p></th>
<th><p>IV size (bytes)</p></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p>Rijndael</p></td>
<td><p>256</p></td>
<td><p>256</p></td>
<td><p>32</p></td>
</tr>
<tr class="even">
<td><p>3DES</p></td>
<td><p>64</p></td>
<td><p>192</p></td>
<td><p>8</p></td>
</tr>
</tbody>
</table>

You need to provide the following information as part of the [Password Protected Package](http://developer.healthvault.com/pages/types/type.aspx?id=c9287326-bb43-4194-858c-8b60768f000f) type when creating the DOPU package:

<table>
<colgroup>
<col width="50%" />
<col width="50%" />
</colgroup>
<thead>
<tr class="header">
<th><p>Property</p></th>
<th><p>Description</p></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p>Encryption algorithm name</p></td>
<td><p>Allowed values:</p>
<p><strong>hmac-sha256-aes256</strong>: Encrypted using the hmac-sha2 pseudorandom and the Rijndael encryption function. This is the recommended encryption algorithm.</p>
<p><strong>hmac-sha1-3des</strong>: Encrypted using the hmac-sha1 pseudorandom and the 3DES encryption function. This algorithm is not recommended. It is currently supported for backwards compatibility but may be removed in the future.</p></td>
</tr>
<tr class="even">
<td><p>Salt</p></td>
<td><p>Salt used when hashing the value. This is application dependent, but to support consistency we suggest that the salt be a base64-encoded series of bytes matching the length requirement of the algorithm. For instance, 64 bytes for sha1 and sha256.</p></td>
</tr>
<tr class="odd">
<td><p>Iteration count</p></td>
<td><p>The number of iterations used when hashing the data.</p></td>
</tr>
<tr class="even">
<td><p>Key length</p></td>
<td><p>The key length in bits used. This depends on the encryption algorithm. Currently the supported values are 192 for hmac-sha1-3des and 256 for hmac-sha256-aes256.</p></td>
</tr>
</tbody>
</table>

Managing connect packages
-------------------------

The user has a limited amount of time to pick up the package (currently four weeks); after that the package will expire and no longer be available. If the user attempts and fails to answer the question correctly several times (currently after three chances), she will not be able to use the package code anymore. In either case, the user will have to contact the application to resend the package.

The package can be picked up only once. If the user selects the wrong record, she will have to request that the application resend the package.

Unlike Patient Connect requests, there is no way to determine if a user has picked up a DOPU package. This is a one-time-only transfer of data, so the application isn't authorized to continue exchanging data with the record.

Application configuration
-------------------------

To use DOPU, you must select the "ConnectPackage" methods when configuring your application in the HealthVault [Application Configuration Center (ACC)](https://config.healthvault-ppe.com). This set of methods is not available for mobile apps.

You do not need an action URL, terms of use, or privacy statement if your application only does DOPU. You can set a custom message for the user upon successfully picking up the package:

<img src="https://i-msdn.sec.s-msft.com/dynimg/IC610886.png" title="Patient Connect Success Message" alt="Patient Connect Success Message" id="IC610886" />
Workflow diagram
----------------

<img src="https://i-msdn.sec.s-msft.com/dynimg/IC610887.png" title="Workflow diagram" alt="Workflow diagram" id="IC610887" />
Code sample using the .NET SDK
------------------------------

Creating a DOPU package:

```cs
private static void CreateDopuPackage(
        string friendlyName,   
        string secretQuestion,    
        string secretAnswer,    
        string patientId,    
        double weight,   
        double height,    
        string imagePath)
{
        // Create an offline connection    
    OfflineWebApplicationConnection connection = new OfflineWebApplicationConnection(        
        ApplicationId,        
        HealthServiceUrl,        
        Guid.Empty /* offlinePersonId */);    
        // Create package parameters    
    ConnectPackageCreationParameters parameters = new ConnectPackageCreationParameters(        
        connection,        
        friendlyName,        
        secretQuestion,        
        secretAnswer,        
        patientId);    
        // Create items   
    Weight weightItem = new Weight(    
        new HealthServiceDateTime(DateTime.Now), new WeightValue(weight));    
        Height heightItem = new Height(height);    
    // Stream a blob    
    BlobStore blobStore = heightItem.GetBlobStore(parameters);    
    using (IO.FileStream stream = new IO.FileStream(imagePath, IO.FileMode.Open))    
    {        
        blobStore.Write("attachment", "image\\jpg", stream);   
    }    
    // Create the package    
    string code = ConnectPackage.Create(        
        parameters,        
        new List<HealthRecordItem>() { weightItem, heightItem });    
    // Email the URL and code to the user   
    SendConnectPackageMail(patientId, code);
}
```