标题：
An error is reported when an oversized file is parsed in GBK encoding mode.
内容：
Version: 2.9.10. When the xmlParseMemory interface is used to parse the oversized file (coded as GBK) in the attachment, an error  Huge input lookup is reported. When the file is encoded as UTF8, no error is reported.
附件：
<?xml version="1.0" encoding="GBK"?><DSPPERFs xmlns="urn:gambol:yang:gambol-TESTAPPNEW">
<DSPPERF>
<CELLTYPE>1007</CELLTYPE>
<CELLID>goomtest-pod-f76d4d4fc-2pldz172-105-1-114__1007__0</CELLID>
<UINT64>20000</UINT64>
<STRING>12345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012</STRING>
<CELLNAME>GOCELL</CELLNAME>
<INDEX>20000</INDEX>
</DSPPERF>
  .............................
</DSPPERFs>


XML_PARSE_NODICT


int main()
{
	int rc = 0;
	const char *version = 0;
	char *a=NULL;
	int t=100;
//	xmlDocPtr pstDoc;
	xmlParserCtxtPtr ctxt=NULL;
	//xmlChar xmlBuff[10000000]={0};
	xmlChar *ptr=NULL;// = xmlBuff;
//	xmlTextReaderPtr reader;
	//ptr=(char *)malloc(100);
	//memset(ptr,0,100);
 
 xmlNodePtr pstRoot;
 xmlDocPtr point=0;
 printf("zhaoqilong fopen start\n");
 FILE* pfin = fopen("/usr1/z30001451/TEST_V3R21C00/sdv/build/parse_error_generate.xml", "r");
 if(!pfin) {printf("zhaoqilong fopen error\n");} 
 else{printf("zhaoqilong success.\n");}
 printf("zhaoqilong fseek start.\n");
 fseek(pfin, 0, SEEK_END);            //将读写位置定位到文件尾
int size = ftell(pfin);              //得到文本文件的字节数
char *pbuf = malloc(size+1);
printf("size is %u",size);
fseek(pfin, 0, SEEK_SET);            //将读写位置定位到文件头
printf("zhaoqilong fseek end.\n");
printf("zhaoqilong fread start.\n");
fread(pbuf, sizeof(char), size, pfin); //将整个文件读入到pbuf所指内存中
printf("zhaoqilong fread end.\n");

#if defined(WITH_REG_MM)
    char *cbbversion = NULL;
#endif
#ifdef LINUX
        int uRet;
        VecSmpS stSmpCfg;
        VecSspS stSspCfg;
#endif

	g_stParserstate.pstRootNode = NULL;
    g_stParserstate.pstParentNode = NULL;
    g_stParserstate.iTextNodeCreated = 0;
#if 0
#ifdef CHK_WITH_ESCCHAR_CALLBACK
    	PTxmlCheckEscapeChar pfCheckEscapeChar = XPC_FuncCheckEscapeChar;
#endif
#endif
#ifdef LINUX
    VecApiInitSmp(&stSmpCfg);
	VecApiInitSsp(&stSspCfg);
	stSspCfg.pfMemAlloc = VecMemAlloc_Test;
	stSspCfg.pfMemFree = VecMemFree_Test;
	stSspCfg.pfMemCmp = VecMemCmp_Test;
	stSmpCfg.pfDbgSendFn = VecDebug_Test;
    uRet = VecApiRegSsp(&stSspCfg);
	if (uRet != VPP_SUCCESS) {
        printf("\nVecApiRegSsp Fails\n");
        return -1;
    }
    uRet = VecApiRegSmp(&stSmpCfg);
	if (uRet != VPP_SUCCESS) {
	    printf("\nVecApiRegSsp Fails\n");
        return -1;
    }
#endif
	RegisterXMLTestStubs();

	// Open the SFT Output File
	SFT_file_ptr = fopen(SFT_output_file_path, "w");
	if(SFT_file_ptr == NULL)
	{
		printf("SFT Output File : %s - Cannot be opened.....", SFT_output_file_path);
		return 0;
	}

	// Open the purify report file
	SFTfailtests = fopen("Purify_Report.txt","w");
	if(SFTfailtests == NULL)
	{
		printf("ERROR: UNABLE TO OPEN Purify Report File...\n");
		return 0;
	}

       open_file_for_output("text.txt");
	//Considering that the file path is updated in path buffer.
	// Open the Call Flow Status File
	fp = fopen("allFlowSCtatus.txt", "w");
	if(fp == NULL)
	{
		printf("ERROR: UNABLE TO OPEN Call Flow Status File...\n");
		return 0;

	}

	#if XML_SFT
		gbMemSftFlag = SFT_TRUE;
		gbSFTEnable = SFT_TRUE;
		gulCounterNumOfFs = 3;
	#endif

	Initialize();
	// Register the Region CBB Memory Manager Stubs
#if defined(WITH_REG_MM)
	cbbversion = CbbApiGetVersion();
	printf("\n CBB Version : %s \n",cbbversion);

	rc = cbbRegCallbacks();
	if (0!= rc)
	{
	    printf("SSP Callback Registaration For RegionMemory Manager Fails\n");
	    return rc;
	}
#endif


	rc = RegisterXmlSSPStubs();
	if(rc != XML_SUCCESS)
	{
		printf("\n\r Error in Registering SSP Functions for XML PARSEER EXITING\n\r");
		return(rc);
	}
#ifdef CHK_WITH_ESCCHAR_CALLBACK
	printf("\n!!!!!!!!!!!!!!!! Check Escape Character Function Registered !!!!!!!!!!!!!!!!!!!\n");
	rc  = xml_RegisterCheckEscapeCharCallback(XPC_FuncCheckEscapeChar);
	if (rc != XML_SUCCESS )
	{
		printf("\n\r Error in Registering xml_RegisterCheckEscapeCharCallback\n\r");

		return rc ;
	}
#endif
	//rc = RegisterXmlSMPStubs();
#ifndef TEST_HERT_MT
	rc = RegisterXmlSMPStubs();
	if(rc != XML_SUCCESS)
	{
		printf("\n\r Error in Registering SMP Functions for XML PARSER !!!!!!!!!!!!!!\n\r");
		printf("\n\r Continuing as SMP is non mandatory\n\r");
	}
#endif

	// rc = xml_RegisterTaskRunFreeCpuCallback(Stub_TaskRunFreeCpuCallback, 2000, 20);
	// if(rc != XML_SUCCESS)
	// {
		// printf("\n Error registering TaskRunFreeCpu Callback\n");
		// return 0;
	// }

    initializeLibxml2();

//	XmlParserCallApiTestCases();

	InitCComponent(TestcaseEntry);

    /* Checking the version */
    version = xmlCompApiGetVersion();




    printf("****************************************************\r\n");
    printf("XML PARSER VERSION: %s\r\n",version);
    printf("XML TEST VERSION: %s\r\n",xmlCompApiGetTestVersion());
    printf("*****************************************************\r\n");

    version = CbbApiGetVersion();

    printf("****************************************************\r\n");
    printf("XML Cache VERSION: %s\r\n",version);
    printf("XML Cache VERSION: %s\r\n",CbbApiGetTestVersion());
    printf("*****************************************************\r\n");

    version = VecApiGetVersion();

    printf("****************************************************\r\n");
    printf("VEC VERSION: %s\r\n",version);
//    printf("VEC VERSION: %s\r\n",CbbApiGetTestVersion());
    printf("*****************************************************\r\n");

//pstDoc = xmlSAXParseMemoryWithData(NULL,buf1,sizeof(buf1),0,NULL);
//xmlDocDumpFormatMemoryEnc(pstDoc,&ptr,&t,"GBK",1);
//xmlFreeDoc(pstDoc);
//xmlFree(ptr);
//ctxt=xmlCreateFileParserCtxt("D:\\libxml_build\\index.xml");
//xmlParseDocument(ctxt);
//pstDoc = xmlSAXParseMemoryWithData(NULL,buf1,sizeof(buf1),1,NULL);
//xmlFreeDoc(pstDoc);
	//ctxt=xmlCreateFileParserCtxt("D:\\libxml_build\\index.xml");
	//xmlCtxtReadFile(ctxt,"D:\\libxml_build\\index.xml",NULL,0);
	//ctxt=xmlCreateFileParserCtxt("D:\\libxml_build\\index.xml");
	//xmlCtxtReadFile(ctxt,"D:\\libxml_build\\index.xml",NULL,1);
	//xmlParseFile("D:\\libxml_build\\index.xml");
	//reader = xmlReaderForFile("D:\\libxml_build\\index.xml","UTF-8",0);
	//xmlTextReaderRead(reader);
	//xmlFreeURI((xmlParseURI(".m‡aa~TTTTTTTTTTTTTTTTTTT€@aa"));
	//Stub_xmlIOParseDTD_Fuzz_1();
	//xmlParseFile("D:\\libxml_build\\index.xml");
	//pstDoc=xmlSAXParseFile(NULL,"D:\\libxml_build\\index.xml",0);
	//xmlFreeDoc(pstDoc);
	//pstDoc=NULL;
	//pstDoc=xmlSAXParseFile(NULL,"D:\\libxml_build\\index.xml",1);
	//xmlFreeDoc(pstDoc);
	//xmlSAXParseEntity(NULL,"D:\\libxml_build\\index.xml");
	//xmlReadFile("D:\\libxml_build\\index.xml",NULL,9999888);
	//Stub_xmlIOParseDTD_Fuzz_1();
	//a=(char *)malloc(10000);
	//memset(a,'a',10000);
	//memset(a,'a',1);
	//memset(a+1,'0',1);
	//xmlParseURI(a);
	//free(a);
	/*if(0 == init_UDP_socket())
    {
        printf("\n The creation of the socket is sucessfull \n");
//Stub_xmlParseDoc2();

        while(!g_Exit)
        {
            read_UDP_msg();
#ifdef WINDOWS
		_sleep(200);
#endif
        }

       printf("\n The Data Recieving is ended\n");
    }
    else
    {
        printf("\n The creation of the socket is Not sucessfull \n");
    }
*/
//    teststream();
    //xmlXPathFreeContext(ctxtXPath);
    //fxmlCleanupParser();
    printf("zhaoqilong xmlParseMemory start.\n");
    xmlKeepBlanksDefault(0);
    printf("length of pbuf before is %d",strlen(pbuf));
    pbuf=OMI_BTrim(pbuf);
    printf("length of pbuf after is %d",strlen(pbuf));
    point=xmlParseMemory(pbuf,size);
    //point=xmlReadMemory(pbuf, size, NULL, "GBK", XML_PARSE_HUGE); 
    if(point==NULL)
    {printf("zhaoqilong xmlParseMemory fail\n");}
   else
   {printf("zhaoqilong xmlParseMemory success\n");
  
    pstRoot =xmlDocGetRootElement(point);
    if(!pstRoot){printf("zhaoqilong xmlDocGetRootElement fail\n");}
    else{printf("zhaoqilong xmlDocGetRootElement success\n");}

  xmlFreeDoc(point);}
    printf("zhaoqilong xmlParseMemory end\n");
    free(pbuf);
    fclose(pfin);
    
    xml_ShutdownXmlParser();
    //xmlMemoryDump();

	/*
	// Open the SFT Output File
	SFT_file_ptr = fopen(SFT_output_file_path, "w");
	if(SFT_file_ptr == NULL)
	{
		printf("SFT Output File : %s - Cannot be opened.....", SFT_output_file_path);
	}
	*/

    return 0;
}
