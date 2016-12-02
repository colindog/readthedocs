Java
=================

如果使用maven，可以加入如下依赖::

	<dependency>
	    <groupId>com.baoquan</groupId>
	    <artifactId>eagle-sdk</artifactId>
	    <version>1.0.8</version>
	</dependency>

如果使用gradle，可以加入如下依赖::
	
	compile group: 'com.baoquan', name: 'eagle-sdk', version: '1.0.8'

初始化客户端
------------------

::

	BaoquanClient client = new BaoquanClient();
	// 设置api地址，比如保全网的测试环境地址
	client.setHost("https://baoquan.com"); 
	// 设置access key
	client.setAccessKey("fsBswNzfECKZH9aWyh47fc"); 
	// 设置rsa私钥文件的绝对路径
	client.setPemPath("path/to/rsa_private.pem"); 

rsa私钥文件应该以 **-----BEGIN PRIVATE KEY-----** 开头和 **-----END PRIVATE KEY-----** 结尾，比如::

	-----BEGIN PRIVATE KEY-----
	MIICeAIBADANBgkqhkiG9w0BAQEFAASCAmIwggJeAgEAAoGBALS8adG98pHSLEq6
	kOT6PG25GMBzpiSs1oXwnPLTOVOYarffF0xSB7nk5yxbqx5BseJNz2NxyTpeJOk8
	FXEI7qTbS6oYAgyH/2HMr5Az3pKGLRdIjJQrpu3qpJkzRw82qGP2MkmVkUYeOl9B
	ZEUpk1GmziwrhbD0zcJITA0mnUqnAgMBAAECgYBnetUPjLTcvrwzURxyrb95hxff
	4JdIuljdOUVzVnKlJUg83JOHVBQuYBvn7thLq4uAqdJK+rQfIhX6IDeaj2WqsO7Y
	d4YoVxFAlfaHIICJKur15KOXuPMpdm3ilZ0c2yCTrJ0m3Xm6mpwd4blDDSupmlj4
	HEXXiInGZgwfTqONAQJBAOlX3EyvE2NvzYMh39wz11fmOi0UiyIvz0immjed4dhV
	0YvPjx8Gj7XGwCkzbuNwr7tlkMTaSiYR8cw1QzV4QoECQQDGSOlgAJC8oUP2+u4H
	+A83jfSLlhQ8XKAJn5Din9kBvs4eKMSjTpJiDBgA7NUAhUfCqS2/m5TiTiS3X3Ij
	ZKknAkEA2iaQCQks4SvnQI9s0FuPGdhdz0ODiCSWb9+CEjkCqdQhococje7+b/0u
	Ldat9uilAlfD7qX96HWiTz4EZXrXAQJBAJ+CbgMl0Ul9bcBUsoHEovEtCEn2TIcW
	eEPlkldNAfSuev+2CiHZhlbLpc+wtdU6YrUNBdl7HjVDabP+W0JvqscCQQDBoUR8
	Y3NUOdGRcaSgwT56tP5J1cZxg1b4vCyr+YfvcEGSBrEaxEugDUjxbON4etMVflh/
	H3QNSvRf4XQ44wQO
	-----END PRIVATE KEY-----

其它初始化设置
^^^^^^^^^^^^^^^

还有些其它可选的初始化设置，比如设置api版本，设置request id生成器，默认情况下你无需进行这些设置::
	
	// 设置api版本
	client.setVersion("v1") 
	// 设置request id生成器，生成器需要实现RequestIdGenerator接口中的createRequestId方法
	client.setRequestIdGenerator(CustomRequestGenerator) 

	// sdk中默认的request id生成器
	public class DefaultRequestIdGenerator implements RequestIdGenerator {

		@Override
		public String createRequestId() {
			return UUID.randomUUID().toString();
		}
	}

客户端初始化完成后即可调用客户端中的方法发送请求

创建保全
------------------

::

	CreateAttestationPayload payload = new CreateAttestationPayload();
	// 设置保全唯一码
	payload.setUniqueId("e68eb8bc-3d7a-4e22-be47-d7999fb40c9a");
	// 设置模板id
	payload.setTemplateId("5Yhus2mVSMnQRXobRJCYgt"); 
	// 设置陈述是否上传完成，如果设置成true，则后续不能继续追加陈述
	payload.setCompleted(false); 
	// 设置保全所有者的身份标识，标识类型定义在IdentityType中
	Map<IdentityType, String> identities = new HashMap<>();
	identities.put(IdentityType.ID, "42012319800127691X");
	identities.put(IdentityType.MO, "15857112383");
	payload.setIdentities(identities);
	// 添加陈述对象列表
	List<Factoid> factoids = new ArrayList<>();
	// 添加product陈述
	Factoid factoid = new Factoid();
	Product product = new Product();
	product.setName("xxx科技有限公司");
	product.setDescription("p2g理财平台");
	factoid.setUnique_id("e13912e2-ccce-47df-997a-9f44eb2c7b6c");
	factoid.setType("product");
	factoid.setData(product);
	factoids.add(factoid);
	// 添加user陈述
	factoid = new Factoid();
	User user = new User();
	user.setName("张三");
	user.setRegistered_at("1466674609");
	user.setUsername("tom");
	user.setPhone_number("13452345987");
	factoid.setUnique_id("5bf54bc4-ec69-4a5d-b6e4-a3f670f795f3");
	factoid.setType("user");
	factoid.setData(user);
	factoids.add(factoid);
	payload.setFactoids(factoids);
	// 调用创建保全接口，如果成功则返回保全号，如果失败则返回失败消息
	try {
		CreateAttestationResponse response = client.createAttestation(payload);
		System.out.println(response.getData().getNo());
	} catch (ServerException e) {
		System.out.println(e.getMessage());
	}

如果创建保全时需要给陈述上传对应的附件::

	// 创建3个附件，每个附件都是ByteArrayBody实例，ContentType必须为DEFAULT_BINARY，并且需要设置filename
	InputStream inputStream0 = getClass().getClassLoader().getResourceAsStream("seal.png");
	ByteArrayBody byteArrayBody0 = new ByteArrayBody(IOUtils.toByteArray(inputStream0), ContentType.DEFAULT_BINARY, "seal.png");
	InputStream inputStream1 = getClass().getClassLoader().getResourceAsStream("seal.png");
	ByteArrayBody byteArrayBody1 = new ByteArrayBody(IOUtils.toByteArray(inputStream1), ContentType.DEFAULT_BINARY, "seal.png");
	InputStream inputStream2 = getClass().getClassLoader().getResourceAsStream("contract.pdf");
	ByteArrayBody byteArrayBody2 = new ByteArrayBody(IOUtils.toByteArray(inputStream2), ContentType.DEFAULT_BINARY, "contract.pdf");
	// 创建附件map，key为factoids中的角标，此处设置factoids中第1个factoid有1个附件，第2个factoid有2两个附件
	Map<String, List<ByteArrayBody>> attachments = new HashMap<>();
	attachments.put("0", Collections.singletonList(byteArrayBody0));
	attachments.put("1", Arrays.asList(byteArrayBody1, byteArrayBody2));
	// 此处省略payload的创建
	try {
		CreateAttestationResponse response = client.createAttestation(payload, attachments);
		System.out.println(response.getData().getNo());
	} catch (ServerException e) {
		System.out.println(e.getMessage());
	}

追加陈述
------------------

::

	AddFactoidsPayload addFactoidsPayload = new AddFactoidsPayload();
	// 设置保全号
	addFactoidsPayload.setAno("7F189BBB5FA1451EA8601D0693E36FE7");
	// 添加陈述对象
	factoids = new ArrayList<>();
	factoid = new Factoid();
	User user = new User();
	user.setName("张三");
	user.setRegistered_at("1466674609");
	user.setUsername("tom");
	user.setPhone_number("13452345987");
	factoid.setUnique_id("5bf54bc4-ec69-4a5d-b6e4-a3f670f795f3");
	factoid.setType("user");
	factoid.setData(user);
	factoids.add(factoid);
	addFactoidsPayload.setFactoids(factoids);
	// 调用追加陈述接口，如果成功则返回的success为true，如果失败则返回失败消息
	try {
		AddFactoidsResponse response = client.addFactoids(addFactoidsPayload);
		System.out.println(response.getData().isSuccess());
	} catch (ServerException e) {
		System.out.println(e.getMessage());
	}

追加陈述的时候同样能为陈述上传附件，跟创建保全为陈述上传附件一样。

创建保全(sha256)
------------------

::

	CreateAttestationPayload payload = new CreateAttestationPayload();
	//模板必须为系统提供的文件HASH模板的子模板。
	payload.setTemplateId("filehash");
	payload.setUniqueId(randomUniqueId());
	Map<IdentityType, String> identities = new HashMap<IdentityType, String>();
	identities.put(IdentityType.MO, "15857110000");
	payload.setIdentities(identities);
	List<Factoid> factoids = new ArrayList<Factoid>();
	Factoid factoid = new Factoid();
	factoid.setUnique_id(randomUniqueId());
	factoid.setType("file");
	Map<String,String> map = new HashMap<String, String>();
	factoid.setData(map);
	map.put("owner_name","李三");
	map.put("owner_id","330124199501017791");
	factoids.add(factoid);
	payload.setFactoids(factoids);
	// 调用创建保全接口，如果成功则返回保全号，如果失败则返回失败消息
	try {
		String sha256 = "654c71176b207401445fdd471f5e023f65af50d7361bf828e5b1c19c89b977b0";
		CreateAttestationResponse response = client.createAttestationWithSha256(payload,sha256);
		System.out.println(response.getData().getNo());
	} catch (ServerException e) {
		System.out.println(e.getMessage());
	}


获取保全数据
------------------

::

	try {
		GetAttestationResponse response = client.getAttestation("DB0C8DB14E3C44C7B9FBBE30EB179241", null);
		System.out.println(response.getData());
	} catch (ServerException e) {
		System.out.println(e.getMessage());
	}	

getAttestation有两个参数，第1个参数ano是保全号，第二个参数fields是一个数组用于设置可选的返回字段

下载保全文件
------------------

::

	try {
		DownloadFile downloadFile = client.downloadAttestation("7FF4E8F6A6764CD0895146581B2B28AA");

		FileOutputStream fileOutputStream = new FileOutputStream(downloadFile.getFileName());
		IOUtils.copy(downloadFile.getFile(), fileOutputStream);
		fileOutputStream.close();
	} catch (ServerException e) {
		System.out.println(e.getMessage());
	}

申请ca证书
------------------

申请个人ca证书::
	
	try {
		ApplyCaPayload payload = new ApplyCaPayload();
		payload.setType(CaType.PERSONAL);
		payload.setLinkName("张三");
		payload.setLinkIdCard("330184198501184115");
		payload.setLinkPhone("13378784545");
		payload.setLinkEmail("123@qq.com");
		ApplyCaResponse response = client.applyCa(payload, null);
		System.out.println(response.getData().getNo());
	} catch (ServerException e) {
		System.out.println(e.getMessage());
	}

三证合一情况，申请企业证书::

	try {
		ApplyCaPayload payload = new ApplyCaPayload();
		payload.setType(CaType.ENTERPRISE);
		payload.setName("xxx有限公司");
		payload.setIcCode("91332406MA27XMXJ27");
		payload.setLinkName("张三");
		payload.setLinkIdCard("330184198501184115");
		payload.setLinkPhone("13378784545");
		payload.setLinkEmail("123@qq.com");
		ApplyCaResponse response = client.applyCa(payload, null);
		System.out.println(response.getData().getNo());
	} catch (ServerException e) {
		System.out.println(e.getMessage());
	}

非三证合一情况，申请企业证书::

	try {
		ApplyCaPayload payload = new ApplyCaPayload();
		payload.setType(CaType.ENTERPRISE);
		payload.setName("xxx有限公司");
		payload.setIcCode("419001000033792");
		payload.setOrgCode("177470403");
		payload.setTaxCode("419001177470403");
		payload.setLinkName("张三");
		payload.setLinkIdCard("330184198501184115");
		payload.setLinkPhone("13378784545");
		payload.setLinkEmail("123@qq.com");
		ApplyCaResponse response = client.applyCa(payload, null);
		System.out.println(response.getData().getNo());
	} catch (ServerException e) {
		System.out.println(e.getMessage());
	}