


### 宏观Chaincode

在fabric生态圈中，chaincode分为系统chaincode和用户chaincode
系统chaincode属于fabric内部的，是在peer启动或者创建channel时完成了register和deploy;
而用户chaincode是外带的/用户可自定义的，由用户去完成一个货币兑换等具体操作，需要用户登录到具体链中的某个peer上进行deploy等操作

##系统chaincode简介及结合SDK的应用

目前系统chaincode分为5种：

    *CSCC*
    *ESCC*
    *QSCC*
    *VSCC*
    *LCCC*

    * **cscc:** Config System Chaincode  实现对peer的配置处理，对于每个来自ordering service的configuration transaction**
    (V1.0中认为channel配置也是一种tx，也需要放到创世区块的ledger中)**，committer都会调用该chaincode的相关方法来处理。
		* Init: 创建chain时，每个chain调用一次；可以在此初始化任何变量
		* Invoke: 有4个方法
			1. `JoinChain`: 处理join chain，由peer发起tx proposal调用，即peer channel join命令指定一个configuration block为参
			数，Chaincode将此block作为chain的genesis block来初始化一个chain。这个genesis block是peer channel create命令创建的
			2. `UpdateConfigBlock`: 更新configuration block，由commmitter调用
			3. `GetConfigBlock`: 获取指定chainID当前的configuration block，由peer调用，如果该peer不属于这个chain，返回error
			4. `GetChannels`: 获取该peer的所有相关channel信息，由peer调用
			现在javaSDK中对已知的排序节点所在的链可以直接获取到configBlock，即已经实现getConfigBlock。
			对于updateConfigBlock还没实现，如果只用于测试环境想把configBlock换掉，那就只能shut down某个chain。
			对于创建一个新的链的时候，直接构造ChainConfiguration对象时指定一个.tx文件进行配置链中channel即可。
			因为它是个流式文件，试用过几个编码打开都不太理想。下面展示打开后比较理想的.tx：

                        �2
                        u
                        Q�ź�"bar*@7cfc9323e70b360ced81758733a367bdbeb6354160e7f2141899cc21ecaa904b
                        certi��;~����4�T��	E�|U�1
                        �0
                        bar�0�
                        Orderer��

                        OrdererMSP��
                        MSP���

                        OrdererMSP�-----BEGIN CERTIFICATE-----
                        MIICKTCCAdCgAwIBAgIRALz4qIofOY8ff94YDATVyGIwCgYIKoZIzj0EAwIwZjEL
                        MAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNhbiBG
                        cmFuY2lzY28xFDASBgNVBAoTC29yZGVyZXJPcmcxMRQwEgYDVQQDEwtvcmRlcmVy
                        T3JnMTAeFw0xNzAzMDExNzM2NDFaFw0yNzAyMjcxNzM2NDFaMGYxCzAJBgNVBAYT
                        AlVTMRMwEQYDVQQIEwpDYWxpZm9ybmlhMRYwFAYDVQQHEw1TYW4gRnJhbmNpc2Nv
                        MRQwEgYDVQQKEwtvcmRlcmVyT3JnMTEUMBIGA1UEAxMLb3JkZXJlck9yZzEwWTAT
                        BgcqhkjOPQIBBggqhkjOPQMBBwNCAARNSaTugowp/Y4XcY7Hrs+m3oE/j8B/jIp3
                        H8thNhYUdkHX69wNsRB6v/vElHn6CPjUHpNAivbXw9dIz7X3aI/Xo18wXTAOBgNV
                        HQ8BAf8EBAMCAaYwDwYDVR0lBAgwBgYEVR0lADAPBgNVHRMBAf8EBTADAQH/MCkG
                        A1UdDgQiBCBNSnciFRaLZZTIfoJlDkOPHzfDA+FLX55vPuBswruCOjAKBggqhkjO
                        PQQDAgNHADBEAiBa6k7Cax+McCHy61Jma1vLuFZswBbnsC6DqbveiKdUoAIgeyAf
                        HzWxMoVrLfPFwF75PqCjae7xnYq+RWlsHZlMGFU=
                        -----END CERTIFICATE-----
                        "�-----BEGIN CERTIFICATE-----
                        MIICKTCCAdCgAwIBAgIRALz4qIofOY8ff94YDATVyGIwCgYIKoZIzj0EAwIwZjEL
                        MAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNhbiBG
                        cmFuY2lzY28xFDASBgNVBAoTC29yZGVyZXJPcmcxMRQwEgYDVQQDEwtvcmRlcmVy
                        T3JnMTAeFw0xNzAzMDExNzM2NDFaFw0yNzAyMjcxNzM2NDFaMGYxCzAJBgNVBAYT
                        AlVTMRMwEQYDVQQIEwpDYWxpZm9ybmlhMRYwFAYDVQQHEw1TYW4gRnJhbmNpc2Nv
                        MRQwEgYDVQQKEwtvcmRlcmVyT3JnMTEUMBIGA1UEAxMLb3JkZXJlck9yZzEwWTAT
                        BgcqhkjOPQIBBggqhkjOPQMBBwNCAARNSaTugowp/Y4XcY7Hrs+m3oE/j8B/jIp3
                        H8thNhYUdkHX69wNsRB6v/vElHn6CPjUHpNAivbXw9dIz7X3aI/Xo18wXTAOBgNV
                        HQ8BAf8EBAMCAaYwDwYDVR0lBAgwBgYEVR0lADAPBgNVHRMBAf8EBTADAQH/MCkG
                        A1UdDgQiBCBNSnciFRaLZZTIfoJlDkOPHzfDA+FLX55vPuBswruCOjAKBggqhkjO
                        PQQDAgNHADBEAiBa6k7Cax+McCHy61Jma1vLuFZswBbnsC6DqbveiKdUoAIgeyAf
                        HzWxMoVrLfPFwF75PqCjae7xnYq+RWlsHZlMGFU=
                        -----END CERTIFICATE-----
                        Admins"3
                        Readers( 

                        OrdererMSPAdmins"3
                        Writers( 

                        OrdererMSPAdmins"4
                        Admins*  

                        OrdererMSPAdmins%
                        CreationPolicy
                        AcceptAllPolicy"
                            BatchSize
                        ���1�� Admins
                        BatchTimeout
                        10sAdmins
                        ChannelRestrictionsAdmins!

                        ConsensusType
                        soloAdmins""
                        Admins

                        AdminsAdmins"*
                        BlockValidation
                        
                        WritersAdmins""
                        Readers
                        
                        ReadersAdmins""
                        Writers
                        
                        WritersAdmins�
                        Application��
                        Org2MSP��
                        MSP���
                        Org2MSP�-----BEGIN CERTIFICATE-----
                        MIICHTCCAcSgAwIBAgIRALakYEdO1ZkArcOQHj85ay8wCgYIKoZIzj0EAwIwYDEL
                        MAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNhbiBG
                        cmFuY2lzY28xETAPBgNVBAoTCHBlZXJPcmcyMREwDwYDVQQDEwhwZWVyT3JnMjAe
                        Fw0xNzAzMDExNzM2NDFaFw0yNzAyMjcxNzM2NDFaMGAxCzAJBgNVBAYTAlVTMRMw
                        EQYDVQQIEwpDYWxpZm9ybmlhMRYwFAYDVQQHEw1TYW4gRnJhbmNpc2NvMREwDwYD
                        VQQKEwhwZWVyT3JnMjERMA8GA1UEAxMIcGVlck9yZzIwWTATBgcqhkjOPQIBBggq
                        hkjOPQMBBwNCAAQrmpN8hPpqLhDFBlyBGOPA1htPscrn091QEqpu3/BPhVHZ0K8g
                        LZpvF/kDsK24uAoovzEyyx6H/RYPl1KIgEHWo18wXTAOBgNVHQ8BAf8EBAMCAaYw
                        DwYDVR0lBAgwBgYEVR0lADAPBgNVHRMBAf8EBTADAQH/MCkGA1UdDgQiBCCWvbV/
                        Tvvc8gGiaYmetH4qc/u3KK4U1H0NWvt13epx3jAKBggqhkjOPQQDAgNHADBEAiAe
                        1/wXZht2Gg6KVxf5lPdAOeoBWZzG0/TQN1KxTH7/QwIgMdJoWxbq2EzihNJlA/U0
                        3+aFesZjYUGvuvOA0ijYFgA=
                        -----END CERTIFICATE-----
                        "�-----BEGIN CERTIFICATE-----
                        MIICHTCCAcSgAwIBAgIRALakYEdO1ZkArcOQHj85ay8wCgYIKoZIzj0EAwIwYDEL
                        MAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNhbiBG
                        cmFuY2lzY28xETAPBgNVBAoTCHBlZXJPcmcyMREwDwYDVQQDEwhwZWVyT3JnMjAe
                        Fw0xNzAzMDExNzM2NDFaFw0yNzAyMjcxNzM2NDFaMGAxCzAJBgNVBAYTAlVTMRMw
                        EQYDVQQIEwpDYWxpZm9ybmlhMRYwFAYDVQQHEw1TYW4gRnJhbmNpc2NvMREwDwYD
                        VQQKEwhwZWVyT3JnMjERMA8GA1UEAxMIcGVlck9yZzIwWTATBgcqhkjOPQIBBggq
                        hkjOPQMBBwNCAAQrmpN8hPpqLhDFBlyBGOPA1htPscrn091QEqpu3/BPhVHZ0K8g
                        LZpvF/kDsK24uAoovzEyyx6H/RYPl1KIgEHWo18wXTAOBgNVHQ8BAf8EBAMCAaYw
                        DwYDVR0lBAgwBgYEVR0lADAPBgNVHRMBAf8EBTADAQH/MCkGA1UdDgQiBCCWvbV/
                        Tvvc8gGiaYmetH4qc/u3KK4U1H0NWvt13epx3jAKBggqhkjOPQQDAgNHADBEAiAe
                        1/wXZht2Gg6KVxf5lPdAOeoBWZzG0/TQN1KxTH7/QwIgMdJoWxbq2EzihNJlA/U0
                        3+aFesZjYUGvuvOA0ijYFgA=
                        -----END CERTIFICATE-----
                        Admins%
                        AnchorPeers


                        peer2�7Admins"1
                        Admins' 
                        
                        Org2MSPAdmins"0
                        Readers% 
                        Org2MSPAdmins"0
                        Writers% 
                        Org2MSPAdmins�
                        Org1MSP�%
                        AnchorPeers


                        peer0�7Admins�
                        MSP���
                        Org1MSP�-----BEGIN CERTIFICATE-----
                        MIICHTCCAcOgAwIBAgIQMnFCpjSdv8WBC9VnEvJ4JTAKBggqhkjOPQQDAjBgMQsw
                        CQYDVQQGEwJVUzETMBEGA1UECBMKQ2FsaWZvcm5pYTEWMBQGA1UEBxMNU2FuIEZy
                        YW5jaXNjbzERMA8GA1UEChMIcGVlck9yZzExETAPBgNVBAMTCHBlZXJPcmcxMB4X
                        DTE3MDMwMTE3MzY0MVoXDTI3MDIyNzE3MzY0MVowYDELMAkGA1UEBhMCVVMxEzAR
                        BgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNhbiBGcmFuY2lzY28xETAPBgNV
                        BAoTCHBlZXJPcmcxMREwDwYDVQQDEwhwZWVyT3JnMTBZMBMGByqGSM49AgEGCCqG
                        SM49AwEHA0IABNeNEm4MKC/uzK+lkxXF5r4aGp8Gal9v3Q9EGXJsNk7zCnL1HD1G
                        xDjw9Fc5gNBgvX1CcFnQYV3VXoJOjMOscY6jXzBdMA4GA1UdDwEB/wQEAwIBpjAP
                        BgNVHSUECDAGBgRVHSUAMA8GA1UdEwEB/wQFMAMBAf8wKQYDVR0OBCIEINils8rB
                        uCH25LSHzq8f0jnNz8MQiUFQkIuQ8F6ReVVqMAoGCCqGSM49BAMCA0gAMEUCIQC0
                        dL7pz5np3hoAaE41n/0c0Tjjs6zVk+zxysz3u9exKwIgBnhrJFK1rV13VUz+W8sp
                        8lrz5ZETok8lPoisXwRIe/E=
                        -----END CERTIFICATE-----
                        "�-----BEGIN CERTIFICATE-----
                        MIICHTCCAcOgAwIBAgIQMnFCpjSdv8WBC9VnEvJ4JTAKBggqhkjOPQQDAjBgMQsw
                        CQYDVQQGEwJVUzETMBEGA1UECBMKQ2FsaWZvcm5pYTEWMBQGA1UEBxMNU2FuIEZy
                        YW5jaXNjbzERMA8GA1UEChMIcGVlck9yZzExETAPBgNVBAMTCHBlZXJPcmcxMB4X
                        DTE3MDMwMTE3MzY0MVoXDTI3MDIyNzE3MzY0MVowYDELMAkGA1UEBhMCVVMxEzAR
                        BgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNhbiBGcmFuY2lzY28xETAPBgNV
                        BAoTCHBlZXJPcmcxMREwDwYDVQQDEwhwZWVyT3JnMTBZMBMGByqGSM49AgEGCCqG
                        SM49AwEHA0IABNeNEm4MKC/uzK+lkxXF5r4aGp8Gal9v3Q9EGXJsNk7zCnL1HD1G
                        xDjw9Fc5gNBgvX1CcFnQYV3VXoJOjMOscY6jXzBdMA4GA1UdDwEB/wQEAwIBpjAP
                        BgNVHSUECDAGBgRVHSUAMA8GA1UdEwEB/wQFMAMBAf8wKQYDVR0OBCIEINils8rB
                        uCH25LSHzq8f0jnNz8MQiUFQkIuQ8F6ReVVqMAoGCCqGSM49BAMCA0gAMEUCIQC0
                        dL7pz5np3hoAaE41n/0c0Tjjs6zVk+zxysz3u9exKwIgBnhrJFK1rV13VUz+W8sp
                        8lrz5ZETok8lPoisXwRIe/E=
                        -----END CERTIFICATE-----
                        Admins"1
                        Admins' 
                        
                        Org1MSPAdmins"0
                        Readers% 
                        Org1MSPAdmins"0
                        Writers% 
                        Org1MSPAdmins""
                        Writers
                        
                        WritersAdmins""
                        Admins

                        AdminsAdmins""
                        Readers
                        
                        ReadersAdmins-
                        OrdererAddresses

                        orderer0:7050Admins&
                        HashingAlgorithm
                        SHA256Admins-
                        BlockDataHashingStructure����Admins""
                        Admins

                        AdminsAdmins"%
                        AcceptAllPolicy Admins""
                        Readers
                        
                        ReadersAdmins""
                        Writers
                        
                        WritersAdmins-

                        cert!�Al�/��`��i2�x��U��p	signature	signature


        当然这个tx文件是可以由configtxgen通过configtx.yaml配置.上面的tx文件对应的configtx.yaml为：
    ---
    ################################################################################
    #
    #   Profile - meant to be used with docker-2orgs-2peerseach-e2e.yml
    #
    #   - Different configuration profiles may be encoded here to be specified
    #   as parameters to the configtxgen tool
    #
    ################################################################################
    Profiles:

        TwoOrgs:
            Orderer:
                <<: *OrdererDefaults
                Organizations:
                    - *OrdererOrg
            Application:
                <<: *ApplicationDefaults
                Organizations:
                    - *Org1
                    - *Org2

    ################################################################################
    #
    #   Section: Organizations
    #
    #   - This section defines the different organizational identities which will
    #   be referenced later in the configuration.
    #
    ################################################################################
    Organizations:

        # SampleOrg defines an MSP using the sampleconfig.  It should never be used
        # in production but may be used as a template for other definitions
        - &OrdererOrg
            # DefaultOrg defines the organization which is used in the sampleconfig
            # of the fabric.git development environment
            Name: OrdererMSP

            # ID to load the MSP definition as
            ID: OrdererMSP

            # MSPDir is the filesystem path which contains the MSP configuration
            #########################################################################
            # FIXME: this path needs to be fixed to point to the actual location of #
            # the project 'fabric-sdk-node' in the file system                      #
            #########################################################################
            MSPDir: /opt/gopath/src/github.com/hyperledger/fabric-sdk-java/src/test/fixture/e2e-2Orgs/crypto-config/ordererOrganizations/ordererOrg1/msp

            # BCCSP (Blockchain crypto provider): Select which crypto implementation or
            # library to use
            BCCSP:
                Default: SW
                SW:
                    Hash: SHA2
                    Security: 256
                    # Location of Key Store. If this is unset, a location will
                    # be chosen using 'MSPDir'/keystore
                    FileKeyStore:
                        KeyStore:

        - &Org1
            # DefaultOrg defines the organization which is used in the sampleconfig
            # of the fabric.git development environment
            Name: Org1MSP

            # ID to load the MSP definition as
            ID: Org1MSP

            # MSPDir is the filesystem path which contains the MSP configuration
            #########################################################################
            # FIXME: this path needs to be fixed to point to the actual location of #
            # the project 'fabric-sdk-node' in the file system                      #
            #########################################################################
            MSPDir: /opt/gopath/src/github.com/hyperledger/fabric-sdk-java/src/test/fixture/e2e-2Orgs/crypto-config/peerOrganizations/peerOrg1/msp/

            # BCCSP (Blockchain crypto provider): Select which crypto implementation or
            # library to use
            BCCSP:
                Default: SW
                SW:
                    Hash: SHA2
                    Security: 256
                    # Location of Key Store. If this is unset, a location will
                    # be chosen using 'MSPDir'/keystore
                    FileKeyStore:
                        KeyStore:

            AnchorPeers:
                # AnchorPeers defines the location of peers which can be used
                # for cross org gossip communication.  Note, this value is only
                # encoded in the genesis block in the Application section context
                - Host: peer0
                  Port: 7051

        - &Org2
            # DefaultOrg defines the organization which is used in the sampleconfig
            # of the fabric.git development environment
            Name: Org2MSP

            # ID to load the MSP definition as
            ID: Org2MSP

            # MSPDir is the filesystem path which contains the MSP configuration
            #########################################################################
            # FIXME: this path needs to be fixed to point to the actual location of #
            # the project 'fabric-sdk-node' in the file system                      #
            #########################################################################
            MSPDir: /opt/gopath/src/github.com/hyperledger/fabric-sdk-java/src/test/fixture/e2e-2Orgs/crypto-config/peerOrganizations/peerOrg2/msp/

            # BCCSP (Blockchain crypto provider): Select which crypto implementation or
            # library to use
            BCCSP:
                Default: SW
                SW:
                    Hash: SHA2
                    Security: 256
                    # Location of Key Store. If this is unset, a location will
                    # be chosen using 'MSPDir'/keystore
                    FileKeyStore:
                        KeyStore:

            AnchorPeers:
                # AnchorPeers defines the location of peers which can be used
                # for cross org gossip communication.  Note, this value is only
                # encoded in the genesis block in the Application section context
                - Host: peer2
                  Port: 7051

    ################################################################################
    #
    #   SECTION: Orderer
    #
    #   - This section defines the values to encode into a config transaction or
    #   genesis block for orderer related parameters
    #
    ################################################################################
    Orderer: &OrdererDefaults

        # Orderer Type: The orderer implementation to start
        # Available types are "solo" and "kafka"
        OrdererType: solo

        Addresses:
            - orderer0:7050

        # Batch Timeout: The amount of time to wait before creating a batch
        BatchTimeout: 10s

        # Batch Size: Controls the number of messages batched into a block
        BatchSize:

            # Max Message Count: The maximum number of messages to permit in a batch
            MaxMessageCount: 10

            # Absolute Max Bytes: The absolute maximum number of bytes allowed for
            # the serialized messages in a batch.
            AbsoluteMaxBytes: 99 MB

            # Preferred Max Bytes: The preferred maximum number of bytes allowed for
            # the serialized messages in a batch. A message larger than the preferred
            # max bytes will result in a batch larger than preferred max bytes.
            PreferredMaxBytes: 512 KB

        Kafka:
            # Brokers: A list of Kafka brokers to which the orderer connects
            # NOTE: Use IP:port notation
            Brokers:
                - orderer0:9092

        # Organizations is the list of orgs which are defined as participants on
        # the orderer side of the network
        Organizations:

    ################################################################################
    #
    #   SECTION: Application
    #
    #   - This section defines the values to encode into a config transaction or
    #   genesis block for application related parameters
    #
    ################################################################################
    Application: &ApplicationDefaults

        # Organizations is the list of orgs which are defined as participants on
        # the application side of the network
        Organizations:

        这里啰嗦一点，重构一个链的时候。需要确定：order节点/普通节点/tx配置文件。
        configtx.yaml需要通过configtxgen生成tx文件





	* **escc:** Endorser System Chaincode  实现背书策略，即为proposal hash和read-write set签名
		* Init: deploy这个chaincode时执行
		* Invoke: 为指定的Proposal 背书签名并返回背书结果，可以自定义更复杂的escc
		**（在user Chaincode deploy或者upgrade时指定对应的escc参数，同时指定背书策略policy参数）** 。
		只是对另一个chaincode的执行结果进行签名背书，绝不修改state。该Chaincode在Endorser的ProcessProposal中被调用。
		一般client会先调用```EndorserClient.ProcessProposal```获取该Chaincode返回的ProposalResponse，然后根据ProposalResponse
		调用 ```putils.CreateSignedTx```创建一个Envelope Tx用```BroadcastClient.Send()```发送到到Ordering service。
		客户端SDK有具体的接口实现背书策略,在调用用户chaincode的init接口或者升级用户chaincode需要指定背书策略。
		现在javaSDK中用 `chaincodeEndorsementPolicy.fromYamlFile（）`方法来实现背书策略，只要在方法里
		输入yaml的路径就行了，yaml的形式如下：
		---
        # A Shotgun policy
        identities:  # list roles to be used in the policy
            user1: {"role": {"name": "member", "mspId": "Org1MSP"}} # role member in org with mspid Org1MSP
            user2: {"role": {"name": "member", "mspId": "Org2MSP"}}
            admin1: {"role": {"name": "admin", "mspId": "Org1MSP"}} # admin role.
            admin2: {"role": {"name": "admin", "mspId": "Org2MSP"}}

        policy: # the policy  .. could have been flat but show grouping.
            1-of: # signed by one of these groups  can be <n>-of  where <n> is any digit 2-of, 3-of etc..
              - 1-of:
                - signed-by: "user1" # a reference to one of the identities defined above.
                - signed-by: "admin1"
              - 1-of:
                - signed-by: "user2"
                - signed-by: "admin2"

         背书节点根据以上背书策略及`sendInstallProposal()`/`sendInstantiationProposal()`确定的peers来定



	* **qscc:** Query System Chaincode  实现查询Chain、Ledger及Tx信息，**只是临时的，以后会将这些方法放到stub中**
		* Init: 略
		* Invoke: 含五个方法`GetChainInfo``GetBlockByNumber``GetBlockByHash` `GetTransactionByID``GetBlockByTxID`
		这几个函数，javaSDK都已经在Chain.java里面实现，他们对应的方法为`queryBlockchainInfo` `queryBlockByNumber`
		`queryBlockByHash` `queryTransactionByID`  `queryBlockTransactionID`



	* **vscc:** Validator System Chaincode 实现默认的验证策略，验证read-write set和背书签名的正确性
		* Init: chaincode启动时调用一次，最好是在Init中什么也不做
		* Invoke: 校验read-write set有效性和至少一个正确的背书，可以自定义更复杂的vscc
		**（在user Chaincode deploy或者upgrade时指定对应的vscc参数）**。在commiter的VSCCValidateTx中被调用。
		节点验证这块javaSDK也已经实现，如`ProposalResponse.java`的`verify()`再调用`CryptoPrimitives`的`verity（）`具体进行对比
		验证由CSCC存储的和提议响应返回的 `plaintext/signature/pemCertificate`
		当然也可以在chaincode deploy或者upgrade sendInstallProposal()指定TransactionContext中verify为false，
		这样上面的验证就不会进行了。


	* **lccc:** Life Cycle System Chaincode 通过该Chaincode的Invoke proposals实现其他Chaincode的lifecycle管理，
	如install、deploy、upgrade、start、stop等。
		* Init: 略
		* Invoke: 有八个方法
			1. `install`：**执行peer chaincode install时，会创建一个lccc.Invoke.install的InstallProposal，调用
			```EndorserClient.ProcessProposal()```来执行Chaincode，即执行lccc中Invoke的install方法
			将ChaincodeDeploymentSpec信息存到```chaincodeInstallPath/chaincodeName.chainVersion```文件中**
			2. `deploy`：**执行peer chaincode instantiate时，会创建一个lccc.Invoke.deploy的DeployProposal，调用
			```EndorserClient.ProcessProposal()```来执行Chaincode，即执行lccc中的Invoke的deploy方法将Chaincode数据放到state中
			（```stub.PutState(ChaincodeName, ChaincodeData)```）,然后再执行命令行指定的Chaincode的deploy，
			具体见`ProcessProposal.simulate`**
			3. `upgrade`：同deploy
			4. `getid`：从lccc的state（GetState）中获取chaincode name
			5. `getdepspec`：从FS中获取ChaincodeDeploymentSpec信息
			6. `getccdata`：从lccc的state（GetState）中获``ChaincodeData{Name, Version, DepSpec, Escc, Vscc, Policy}``
			7. `getchaincodes`：从lccc的state（GetStateByRange）中获取`ChaincodeInfo{Name, Version, Path, Input, Escc, Vscc}`
			8. `getinstalledchaincodes`：从FS中获取所有install的Chaincode
		javaSDK对应上面方法为：
		    1.sendInstallProposal
		    2.sendInstantiationProposal
		    3.sendUpgradeProposal
		    4.getChaincodeName
		    5.6.：分散的getChaincodeName/getChaincodeVersion/getChaincodeID/getChaincodeLanguage/getChaincodeEndorsementPolicy
		    7.getArgs,暂时未见获取 vscc/escc
		    8.暂时未实现




##用户chaincode简介

    用户chaincode指的是由用户自己定义的，在sdk的辅助下（主要是通过上面的系统chaincode及其他sdk函数)
    通过调用fabric/core/chaincode/shim/java下的源码接口,完成用户所需要的创建币/分发币/交换币/记录操作流程等一系列操作。
    （这个目录下的接口也可以自己定义）
    具体编写javaChaincode时，只需要继承抽象类 `fabric/shim/ChaincodeBase` 就可以，`ChaincodeBase`继承自接口类 `Chaincode`
    用户Chaincode必须实现其继承类`ChaincodeBase`的两个抽象方法：

       `public Response init(ChaincodeStub stub);`
       `public Response invoke(ChaincodeStub stub);`




