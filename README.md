export interface ApiResponseType {
  jobDetails?: {
    JOB_ID?: string;
    JOB_NAME?: string;
    DESCRIPTION?: string;
    JOB_TYPE?: string;
    JOB_CATEGORY?: string;
    ACTIVE_FLAG?: string;
  };
  operationConfigurationDetails?: [
    {
      OPERATION_ID?: string;
      OPERATION_NAME?: string;
      OPERATION_TYPE?: string;
      OPERATION_TAG?: string;
      OPERATION_TAG_VALUE?: string;
      OPERATION_TAG_VALUE_GUI?: string;
    }
  ];
}

  useEffect(() => {
    const formattedRows: { string: string; id: number }[] = jobContext?.jobData?.operationConfigRows?.map((row?, index?) => ({
      id: index,
      ...row
    }));
    setRows(formattedRows);
  }, [jobContext?.jobData?.operationConfigRows]);

  const mapJobData = (data?: ApiResponseType) => {
    const apiResponse = data;
    console.log('apiResp', apiResponse.jobDetails.JOB_ID);
    const jobName = apiResponse?.jobDetails?.JOB_NAME || '';
    const jobDescription = apiResponse?.jobDetails?.DESCRIPTION || '';
    const jobType = apiResponse?.jobDetails?.JOB_TYPE || '';

    const operationConfigRows: { string: string }[] = apiResponse?.map((item?) => ({
      operationSequence: item?.operationConfigurationDetails?.OPERATION_ID,
      operationName: item?.operationConfigurationDetails?.OPERATION_NAME,
      operationTag: item?.operationConfigurationDetails?.OPERATION_TAG,
      operationTagValue: item?.operationConfigurationDetails?.OPERATION_TAG_VALUE_GUI ?? item?.operationConfigurationDetails?.OPERATION_TAG_VALUE,
      operationType: item?.operationConfigurationDetails?.OPERATION_TYPE
    }));
    console.log('rows', operationConfigRows);

    jobContext?.updateJobData({
      jobName,
      jobDescription,
      jobType,
      operationConfigRows
    });
    appContext?.updateIsLoading(false);
  };

response from API:

{
    "jobDetails": {
        "JOB_ID": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "JOB_NAME": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "DESCRIPTION": "Job to load SAP_JP data from S3 files to landing",
        "JOB_TYPE": "snowsql",
        "JOB_CATEGORY": "data processing",
        "ACTIVE_FLAG": "Y"
    },
    "operationConfigurationDetails": [
        {
            "OPERATION_ID": "2",
            "OPERATION_NAME": "execute_query",
            "operation_type": "S3_TO_LND_COPY_COMMAND",
            "OPERATION_TYPE": "query",
            "OPERATION_TAG_VALUE": "COPY INTO MED_PTCDH_DEV.MED_PTCDH_LND.LND_SAP_JP_TRANSACTION\n(SRC_SYS_ID ,VENDOR_ID ,VENDOR_NAME1 ,VENDOR_NAME2 ,VENDOR_STREET ,VENDOR_CITY ,VENDOR_STATE ,VENDOR_POSTAL_CODE ,VENDOR_COUNTRY ,VENDOR_COUNTRY_DESCRIPTION ,VENDOR_TELEPHONE ,DOC_NUMBER ,LINE_ITEM ,COMPANY_CODE ,COMPANY_CODE_DESC ,FDM_DIVISION ,FDM_DIVISION_DESCRIPTION ,FDM_MARKET ,FDM_MARKET_DESCRIPTION ,GL_ACCOUNT ,GL_ACCOUNT_DESCRIPTION ,COST_CENTER ,COST_CENTER_DESCRIPTION ,PROFIT_CENTER ,PROFIT_CENTER_DESCRIPTION ,WBS_ELEMENT ,WBS_ELEMENT_DESCRIPTION ,POSTING_DATE ,PAYMENT_METHOD ,PAYMENT_METHOD_DESCRIPTION ,CHECK_ENCASHMENT_DATE ,HCP_FIRST_NAME ,HCP_MIDDLE_NAME ,HCP_LAST_NAME ,HCP_PFIZER_ID ,TRAVEL_CITY ,TRAVEL_STATE ,TRAVEL_COUNTRY ,ARIBA_PUR_REQ_TITLE ,ARIBA_EPAY_TITLE ,AP_DOC_HEADER_TEXT ,CHECK_DESCRIPTION ,CHECK_ISSUE_DATE ,CHECK_NUMBER ,CLEARING_DATE ,CLEARING_DOC_NUMBER ,DOCUMENT_TYPE ,DOCUMENT_TYPE_DESCRIPTION ,FISCAL_PERIOD ,FISCAL_YEAR_VARIANT ,HCP_CREDENTIALS ,HCP_REQ_IND_DETAIL ,INDUSTRY_CODE ,INDUSTRY_CODE_DESCRIPTION ,INSTITUTION_CODE ,INV_CREATED_ON ,INVOICE_DATE ,INVOICE_SOURCE ,MATERIAL_CATEGORY ,MATERIAL_GROUP ,MATERIAL_GROUP_DESCRIPTION ,NET_DUE_DATE ,PLANT_CODE ,PLANT_NUMBER ,PO_LINE_ITEM ,PO_NUMBER ,PURCH_DOC_TYPE ,PO_REQUISITIONER ,TIG_DESCRIPTION1 ,TIG_DESCRIPTION2 ,TIG_DESCRIPTION3 ,TIG_DESCRIPTION4 ,TIG_DESCRIPTION5 ,TIG_REFERENCE_NUMBER ,VENDOR_INVOICE_REFERENCE ,VN_VENDOR_REFERENCE ,AMOUNT_IN_LOCAL_CURRENCY ,AMOUNT_IN_USD ,WP_NUMBER ,LINE_ITEM_DESCRIPTION ,CREATE_DT ,CREATED_BY ,UPDATE_DT ,UPDATED_BY ,FILE_NAME ,LOAD_TIMESTAMP ,DAG_RUN_ID ,JOB_RUN_ID)\nFROM\n(\n\nSELECT\n       ('SAP_JP_FY'|| '-' || SUBSTR(TO_CHAR(LTRIM(RTRIM(a.$48))),11,4) || '-' || TO_CHAR(LTRIM(LTRIM(RTRIM(a.$45)),'0')) || '-' ||'00'|| TO_CHAR((a.$12))  || '-' || TO_CHAR(RTRIM(LTRIM(a.$13,'0')))) as src_sys_id\n,a.$1\n,a.$2\n,a.$3\n,a.$4\n,a.$5\n,a.$6\n,a.$7\n,a.$8\n,a.$9\n,a.$10\n,a.$11\n,a.$12\n,a.$13\n,a.$14\n,a.$15\n,a.$16\n,a.$17\n,a.$18\n,a.$19\n,a.$20\n,a.$21\n,a.$22\n,a.$23\n,a.$24\n,a.$25\n,a.$26\n,a.$27\n,a.$28\n,a.$29\n,a.$30\n,a.$31\n,a.$32\n,a.$33\n,a.$34\n,a.$35\n,a.$36\n,a.$37\n,a.$38\n,a.$39\n,a.$40\n,a.$41\n,a.$42\n,a.$43\n,a.$44\n,a.$45\n,a.$46\n,a.$47\n,a.$48\n,a.$49\n,a.$50\n,a.$51\n,a.$52\n,a.$53\n,a.$54\n,a.$55\n,a.$56\n,a.$57\n,a.$58\n,a.$59\n,a.$60\n,a.$61\n,a.$62\n,a.$63\n,a.$64\n,a.$65\n,a.$66\n,a.$67\n,a.$68\n,a.$69\n,a.$70\n,a.$71\n,a.$72\n,a.$73\n,a.$74\n,a.$75\n,a.$76\n,a.$77\n,a.$78\n,a.$79\n,CURRENT_TIMESTAMP as CREATE_DT\n,'SYSTEM' as CREATED_BY\n,CURRENT_TIMESTAMP as UPDATE_DT\n,'SYSTEM' as UPDATED_BY\n,SPLIT_PART(METADATA$FILENAME, '/', 4) AS FILE_NAME\n,to_time(SPLIT_PART(CURRENT_TIMESTAMP, ' ', 2)) AS LOAD_TIMESTAMP\n,'SAP_WORKFLOW' AS DAG_RUN_ID\n,'SAP_PAYMENT_JP_S3_TO_LANDING' AS JOB_RUN_ID\nfrom \n        @\"MED_PTCDH_DEV\".\"MED_PTCDH_LND\".CDH_LANDING_SAP_PAYMENTS_JP a\n\t\t);",
            "OPERATION_TAG_VALUE_GUI": null
        },
        {
            "OPERATION_ID": "3",
            "OPERATION_NAME": "execute_query",
            "operation_type": "S3_TO_LND_PARAMETERS",
            "OPERATION_TYPE": "TARGET_TABLE_NAME",
            "OPERATION_TAG_VALUE": "LND_SAP_JP_TRANSACTION",
            "OPERATION_TAG_VALUE_GUI": null
        },
        {
            "OPERATION_ID": "3",
            "OPERATION_NAME": "execute_query",
            "operation_type": "S3_TO_LND_PARAMETERS",
            "OPERATION_TYPE": "SOURCE_FILE_NAME",
            "OPERATION_TAG_VALUE": "ZOH_HCP_TP_EXTRACT_PT_ID.DAT",
            "OPERATION_TAG_VALUE_GUI": null
        },
        {
             "OPERATION_ID": "4",
            "OPERATION_NAME": "execute_query",
            "operation_type": "S3_TO_LND_FILE_ARCHIVE",
            "OPERATION_TYPE": "query",
            "OPERATION_TAG_VALUE": "CALL \"MED_PTCDH_DEV\".\"MED_PTCDH_LND\".SP_S3_FILE_ARCHIVE('SAP_PAYMENT_JP_S3_TO_LANDING')",
            "OPERATION_TAG_VALUE_GUI": null
        }
}
