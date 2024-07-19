import React, { useState, useEffect, useContext } from 'react';
import axios from 'axios';

// Assuming jobContext is already created and provides jobData and setJobData
import { JobContext } from './jobContext';

const fetchAndMapJobData = async () => {
  const { jobData, setJobData } = useContext(JobContext);

  try {
    const response = await axios.get('/api/jobData');
    const apiResponse = response.data;

    const jobName = apiResponse[0]?.JOB_NAME || '';
    const jobDescription = apiResponse[0]?.DESCRIPTION || '';
    const jobType = apiResponse[0]?.JOB_TYPE || '';

    const operationConfigRows = apiResponse.map((item, index) => ({
      operationName: item.OPERATION_NAME,
      operationTag: item.OPERATION_TAG,
      operationTagValue: item.OPERATION_TAG_VALUE,
      operationType: item.OPERATION_TYPE,
      operationSequence: index + 1, // Assuming sequence is based on order
    }));

    setJobData({
      jobName,
      jobDescription,
      jobType,
      operationConfigRows,
      viewJobId: apiResponse[0]?.JOB_ID || '',
    });
  } catch (error) {
    console.error('Failed to fetch job data:', error);
  }
};

// Call this function within a useEffect or any other lifecycle method
useEffect(() => {
  fetchAndMapJobData();
}, []);
    {
        "JOB_ID": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "JOB_NAME": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "DESCRIPTION": "Job to load SAP_JP data from S3 files to landing",
        "JOB_TYPE": "snowsql",
        "JOB_CATEGORY": "data processing",
        "ACTIVE_FLAG": "Y",
        "OPERATION_ID": "3",
        "OPERATION_NAME": "execute_query",
        "OPERATION_TYPE": "S3_TO_LND_PARAMETERS",
        "OPERATION_TAG": "TARGET_PATH",
        "OPERATION_TAG_VALUE": "MED_PTCDH_DEV.MED_PTCDH_LND",
        "OPERATION_TAG_VALUE_GUI": null
    },
    {
        "JOB_ID": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "JOB_NAME": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "DESCRIPTION": "Job to load SAP_JP data from S3 files to landing",
        "JOB_TYPE": "snowsql",
        "JOB_CATEGORY": "data processing",
        "ACTIVE_FLAG": "Y",
        "OPERATION_ID": "3",
        "OPERATION_NAME": "execute_query",
        "OPERATION_TYPE": "S3_TO_LND_PARAMETERS",
        "OPERATION_TAG": "SOURCE_PATH",
        "OPERATION_TAG_VALUE": "s3://med-ptcdh-devamrasp20240408124911/SAP/JP/landing/ZOH_HCP_TP_EXTRACT_PT_ID.DAT",
        "OPERATION_TAG_VALUE_GUI": null
    },
    {
        "JOB_ID": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "JOB_NAME": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "DESCRIPTION": "Job to load SAP_JP data from S3 files to landing",
        "JOB_TYPE": "snowsql",
        "JOB_CATEGORY": "data processing",
        "ACTIVE_FLAG": "Y",
        "OPERATION_ID": "3",
        "OPERATION_NAME": "execute_query",
        "OPERATION_TYPE": "S3_TO_LND_JOB_AUDIT_LOG",
        "OPERATION_TAG": "query",
        "OPERATION_TAG_VALUE": "CALL \"MED_PTCDH_TEST\".\"MED_PTCDH_LND\".SP_JOB_RUN_AUDIT_LOGGING('Currency_Conversion_DS_S3_TO_LANDING',3)",
        "OPERATION_TAG_VALUE_GUI": null
    },
    {
        "JOB_ID": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "JOB_NAME": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "DESCRIPTION": "Job to load SAP_JP data from S3 files to landing",
        "JOB_TYPE": "snowsql",
        "JOB_CATEGORY": "data processing",
        "ACTIVE_FLAG": "Y",
        "OPERATION_ID": "3",
        "OPERATION_NAME": "execute_query",
        "OPERATION_TYPE": "S3_TO_LND_PARAMETERS",
        "OPERATION_TAG": "TARGET_TABLE_NAME",
        "OPERATION_TAG_VALUE": "LND_SAP_JP_TRANSACTION",
        "OPERATION_TAG_VALUE_GUI": null
    },
    {
        "JOB_ID": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "JOB_NAME": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "DESCRIPTION": "Job to load SAP_JP data from S3 files to landing",
        "JOB_TYPE": "snowsql",
        "JOB_CATEGORY": "data processing",
        "ACTIVE_FLAG": "Y",
        "OPERATION_ID": "3",
        "OPERATION_NAME": "execute_query",
        "OPERATION_TYPE": "S3_TO_LND_PARAMETERS",
        "OPERATION_TAG": "SOURCE_FILE_NAME",
        "OPERATION_TAG_VALUE": "ZOH_HCP_TP_EXTRACT_PT_ID.DAT",
        "OPERATION_TAG_VALUE_GUI": null
    },
    {
        "JOB_ID": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "JOB_NAME": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "DESCRIPTION": "Job to load SAP_JP data from S3 files to landing",
        "JOB_TYPE": "snowsql",
        "JOB_CATEGORY": "data processing",
        "ACTIVE_FLAG": "Y",
        "OPERATION_ID": "3",
        "OPERATION_NAME": "execute_query",
        "OPERATION_TYPE": "S3_TO_LND_PARAMETERS",
        "OPERATION_TAG": "TARGET_PATH",
        "OPERATION_TAG_VALUE": "MED_PTCDH_DEV.MED_PTCDH_LND",
        "OPERATION_TAG_VALUE_GUI": null
    },
    {
        "JOB_ID": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "JOB_NAME": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "DESCRIPTION": "Job to load SAP_JP data from S3 files to landing",
        "JOB_TYPE": "snowsql",
        "JOB_CATEGORY": "data processing",
        "ACTIVE_FLAG": "Y",
        "OPERATION_ID": "3",
        "OPERATION_NAME": "execute_query",
        "OPERATION_TYPE": "S3_TO_LND_PARAMETERS",
        "OPERATION_TAG": "SOURCE_PATH",
        "OPERATION_TAG_VALUE": "s3://med-ptcdh-devamrasp20240408124911/SAP/JP/landing/ZOH_HCP_TP_EXTRACT_PT_ID.DAT",
        "OPERATION_TAG_VALUE_GUI": null
    },
    {
        "JOB_ID": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "JOB_NAME": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "DESCRIPTION": "Job to load SAP_JP data from S3 files to landing",
        "JOB_TYPE": "snowsql",
        "JOB_CATEGORY": "data processing",
        "ACTIVE_FLAG": "Y",
        "OPERATION_ID": "3",
        "OPERATION_NAME": "execute_query",
        "OPERATION_TYPE": "S3_TO_LND_JOB_AUDIT_LOG",
        "OPERATION_TAG": "query",
        "OPERATION_TAG_VALUE": "CALL \"MED_PTCDH_TEST\".\"MED_PTCDH_LND\".SP_JOB_RUN_AUDIT_LOGGING('Currency_Conversion_DS_S3_TO_LANDING',3)",
        "OPERATION_TAG_VALUE_GUI": null
    },
    {
        "JOB_ID": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "JOB_NAME": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "DESCRIPTION": "Job to load SAP_JP data from S3 files to landing",
        "JOB_TYPE": "snowsql",
        "JOB_CATEGORY": "data processing",
        "ACTIVE_FLAG": "Y",
        "OPERATION_ID": "3",
        "OPERATION_NAME": "execute_query",
        "OPERATION_TYPE": "S3_TO_LND_PARAMETERS",
        "OPERATION_TAG": "TARGET_TABLE_NAME",
        "OPERATION_TAG_VALUE": "LND_SAP_JP_TRANSACTION",
        "OPERATION_TAG_VALUE_GUI": null
    },
    {
        "JOB_ID": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "JOB_NAME": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "DESCRIPTION": "Job to load SAP_JP data from S3 files to landing",
        "JOB_TYPE": "snowsql",
        "JOB_CATEGORY": "data processing",
        "ACTIVE_FLAG": "Y",
        "OPERATION_ID": "3",
        "OPERATION_NAME": "execute_query",
        "OPERATION_TYPE": "S3_TO_LND_PARAMETERS",
        "OPERATION_TAG": "SOURCE_FILE_NAME",
        "OPERATION_TAG_VALUE": "ZOH_HCP_TP_EXTRACT_PT_ID.DAT",
        "OPERATION_TAG_VALUE_GUI": null
    },
    {
        "JOB_ID": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "JOB_NAME": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "DESCRIPTION": "Job to load SAP_JP data from S3 files to landing",
        "JOB_TYPE": "snowsql",
        "JOB_CATEGORY": "data processing",
        "ACTIVE_FLAG": "Y",
        "OPERATION_ID": "3",
        "OPERATION_NAME": "execute_query",
        "OPERATION_TYPE": "S3_TO_LND_PARAMETERS",
        "OPERATION_TAG": "TARGET_PATH",
        "OPERATION_TAG_VALUE": "MED_PTCDH_DEV.MED_PTCDH_LND",
        "OPERATION_TAG_VALUE_GUI": null
    },
    {
        "JOB_ID": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "JOB_NAME": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "DESCRIPTION": "Job to load SAP_JP data from S3 files to landing",
        "JOB_TYPE": "snowsql",
        "JOB_CATEGORY": "data processing",
        "ACTIVE_FLAG": "Y",
        "OPERATION_ID": "3",
        "OPERATION_NAME": "execute_query",
        "OPERATION_TYPE": "S3_TO_LND_PARAMETERS",
        "OPERATION_TAG": "SOURCE_PATH",
        "OPERATION_TAG_VALUE": "s3://med-ptcdh-devamrasp20240408124911/SAP/JP/landing/ZOH_HCP_TP_EXTRACT_PT_ID.DAT",
        "OPERATION_TAG_VALUE_GUI": null
    },
    {
        "JOB_ID": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "JOB_NAME": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "DESCRIPTION": "Job to load SAP_JP data from S3 files to landing",
        "JOB_TYPE": "snowsql",
        "JOB_CATEGORY": "data processing",
        "ACTIVE_FLAG": "Y",
        "OPERATION_ID": "3",
        "OPERATION_NAME": "execute_query",
        "OPERATION_TYPE": "S3_TO_LND_JOB_AUDIT_LOG",
        "OPERATION_TAG": "query",
        "OPERATION_TAG_VALUE": "CALL \"MED_PTCDH_TEST\".\"MED_PTCDH_LND\".SP_JOB_RUN_AUDIT_LOGGING('Currency_Conversion_DS_S3_TO_LANDING',3)",
        "OPERATION_TAG_VALUE_GUI": null
    },
    {
        "JOB_ID": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "JOB_NAME": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "DESCRIPTION": "Job to load SAP_JP data from S3 files to landing",
        "JOB_TYPE": "snowsql",
        "JOB_CATEGORY": "data processing",
        "ACTIVE_FLAG": "Y",
        "OPERATION_ID": "3",
        "OPERATION_NAME": "execute_query",
        "OPERATION_TYPE": "S3_TO_LND_PARAMETERS",
        "OPERATION_TAG": "TARGET_TABLE_NAME",
        "OPERATION_TAG_VALUE": "LND_SAP_JP_TRANSACTION",
        "OPERATION_TAG_VALUE_GUI": null
    },
    {
        "JOB_ID": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "JOB_NAME": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "DESCRIPTION": "Job to load SAP_JP data from S3 files to landing",
        "JOB_TYPE": "snowsql",
        "JOB_CATEGORY": "data processing",
        "ACTIVE_FLAG": "Y",
        "OPERATION_ID": "3",
        "OPERATION_NAME": "execute_query",
        "OPERATION_TYPE": "S3_TO_LND_PARAMETERS",
        "OPERATION_TAG": "SOURCE_FILE_NAME",
        "OPERATION_TAG_VALUE": "ZOH_HCP_TP_EXTRACT_PT_ID.DAT",
        "OPERATION_TAG_VALUE_GUI": null
    },
    {
        "JOB_ID": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "JOB_NAME": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "DESCRIPTION": "Job to load SAP_JP data from S3 files to landing",
        "JOB_TYPE": "snowsql",
        "JOB_CATEGORY": "data processing",
        "ACTIVE_FLAG": "Y",
        "OPERATION_ID": "3",
        "OPERATION_NAME": "execute_query",
        "OPERATION_TYPE": "S3_TO_LND_PARAMETERS",
        "OPERATION_TAG": "TARGET_PATH",
        "OPERATION_TAG_VALUE": "MED_PTCDH_DEV.MED_PTCDH_LND",
        "OPERATION_TAG_VALUE_GUI": null
    },
    {
        "JOB_ID": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "JOB_NAME": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "DESCRIPTION": "Job to load SAP_JP data from S3 files to landing",
        "JOB_TYPE": "snowsql",
        "JOB_CATEGORY": "data processing",
        "ACTIVE_FLAG": "Y",
        "OPERATION_ID": "3",
        "OPERATION_NAME": "execute_query",
        "OPERATION_TYPE": "S3_TO_LND_PARAMETERS",
        "OPERATION_TAG": "SOURCE_PATH",
        "OPERATION_TAG_VALUE": "s3://med-ptcdh-devamrasp20240408124911/SAP/JP/landing/ZOH_HCP_TP_EXTRACT_PT_ID.DAT",
        "OPERATION_TAG_VALUE_GUI": null
    },
    {
        "JOB_ID": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "JOB_NAME": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "DESCRIPTION": "Job to load SAP_JP data from S3 files to landing",
        "JOB_TYPE": "snowsql",
        "JOB_CATEGORY": "data processing",
        "ACTIVE_FLAG": "Y",
        "OPERATION_ID": "3",
        "OPERATION_NAME": "execute_query",
        "OPERATION_TYPE": "S3_TO_LND_JOB_AUDIT_LOG",
        "OPERATION_TAG": "query",
        "OPERATION_TAG_VALUE": "CALL \"MED_PTCDH_TEST\".\"MED_PTCDH_LND\".SP_JOB_RUN_AUDIT_LOGGING('Currency_Conversion_DS_S3_TO_LANDING',3)",
        "OPERATION_TAG_VALUE_GUI": null
    },
    {
        "JOB_ID": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "JOB_NAME": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "DESCRIPTION": "Job to load SAP_JP data from S3 files to landing",
        "JOB_TYPE": "snowsql",
        "JOB_CATEGORY": "data processing",
        "ACTIVE_FLAG": "Y",
        "OPERATION_ID": "4",
        "OPERATION_NAME": "execute_query",
        "OPERATION_TYPE": "S3_TO_LND_FILE_ARCHIVE",
        "OPERATION_TAG": "query",
        "OPERATION_TAG_VALUE": "CALL \"MED_PTCDH_DEV\".\"MED_PTCDH_LND\".SP_S3_FILE_ARCHIVE('SAP_PAYMENT_JP_S3_TO_LANDING')",
        "OPERATION_TAG_VALUE_GUI": null
    },
    {
        "JOB_ID": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "JOB_NAME": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "DESCRIPTION": "Job to load SAP_JP data from S3 files to landing",
        "JOB_TYPE": "snowsql",
        "JOB_CATEGORY": "data processing",
        "ACTIVE_FLAG": "Y",
        "OPERATION_ID": "4",
        "OPERATION_NAME": "execute_query",
        "OPERATION_TYPE": "S3_TO_LND_PARAMETERS",
        "OPERATION_TAG": "EXT_STAGE",
        "OPERATION_TAG_VALUE": "@\"MED_PTCDH_DEV\".\"MED_PTCDH_LND\".CDH_LANDING_SAP_PAYMENTS_JP",
        "OPERATION_TAG_VALUE_GUI": null
    },
    {
        "JOB_ID": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "JOB_NAME": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "DESCRIPTION": "Job to load SAP_JP data from S3 files to landing",
        "JOB_TYPE": "snowsql",
        "JOB_CATEGORY": "data processing",
        "ACTIVE_FLAG": "Y",
        "OPERATION_ID": "4",
        "OPERATION_NAME": "execute_query",
        "OPERATION_TYPE": "S3_TO_LND_PARAMETERS",
        "OPERATION_TAG": "EXT_STAGE_ARCH",
        "OPERATION_TAG_VALUE": "@\"MED_PTCDH_DEV\".\"MED_PTCDH_LND\".CDH_ARCHIVE_SAP_PAYMENTS_JP",
        "OPERATION_TAG_VALUE_GUI": null
    },
    {
        "JOB_ID": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "JOB_NAME": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "DESCRIPTION": "Job to load SAP_JP data from S3 files to landing",
        "JOB_TYPE": "snowsql",
        "JOB_CATEGORY": "data processing",
        "ACTIVE_FLAG": "Y",
        "OPERATION_ID": "4",
        "OPERATION_NAME": "execute_query",
        "OPERATION_TYPE": "S3_TO_LND_FILE_ARCHIVE",
        "OPERATION_TAG": "query",
        "OPERATION_TAG_VALUE": "CALL \"MED_PTCDH_DEV\".\"MED_PTCDH_LND\".SP_S3_FILE_ARCHIVE('SAP_PAYMENT_JP_S3_TO_LANDING')",
        "OPERATION_TAG_VALUE_GUI": null
    },
    {
        "JOB_ID": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "JOB_NAME": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "DESCRIPTION": "Job to load SAP_JP data from S3 files to landing",
        "JOB_TYPE": "snowsql",
        "JOB_CATEGORY": "data processing",
        "ACTIVE_FLAG": "Y",
        "OPERATION_ID": "4",
        "OPERATION_NAME": "execute_query",
        "OPERATION_TYPE": "S3_TO_LND_PARAMETERS",
        "OPERATION_TAG": "EXT_STAGE",
        "OPERATION_TAG_VALUE": "@\"MED_PTCDH_DEV\".\"MED_PTCDH_LND\".CDH_LANDING_SAP_PAYMENTS_JP",
        "OPERATION_TAG_VALUE_GUI": null
    },
    {
        "JOB_ID": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "JOB_NAME": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "DESCRIPTION": "Job to load SAP_JP data from S3 files to landing",
        "JOB_TYPE": "snowsql",
        "JOB_CATEGORY": "data processing",
        "ACTIVE_FLAG": "Y",
        "OPERATION_ID": "4",
        "OPERATION_NAME": "execute_query",
        "OPERATION_TYPE": "S3_TO_LND_PARAMETERS",
        "OPERATION_TAG": "EXT_STAGE_ARCH",
        "OPERATION_TAG_VALUE": "@\"MED_PTCDH_DEV\".\"MED_PTCDH_LND\".CDH_ARCHIVE_SAP_PAYMENTS_JP",
        "OPERATION_TAG_VALUE_GUI": null
    },
    {
        "JOB_ID": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "JOB_NAME": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "DESCRIPTION": "Job to load SAP_JP data from S3 files to landing",
        "JOB_TYPE": "snowsql",
        "JOB_CATEGORY": "data processing",
        "ACTIVE_FLAG": "Y",
        "OPERATION_ID": "4",
        "OPERATION_NAME": "execute_query",
        "OPERATION_TYPE": "S3_TO_LND_FILE_ARCHIVE",
        "OPERATION_TAG": "query",
        "OPERATION_TAG_VALUE": "CALL \"MED_PTCDH_DEV\".\"MED_PTCDH_LND\".SP_S3_FILE_ARCHIVE('SAP_PAYMENT_JP_S3_TO_LANDING')",
        "OPERATION_TAG_VALUE_GUI": null
    },
    {
        "JOB_ID": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "JOB_NAME": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "DESCRIPTION": "Job to load SAP_JP data from S3 files to landing",
        "JOB_TYPE": "snowsql",
        "JOB_CATEGORY": "data processing",
        "ACTIVE_FLAG": "Y",
        "OPERATION_ID": "4",
        "OPERATION_NAME": "execute_query",
        "OPERATION_TYPE": "S3_TO_LND_PARAMETERS",
        "OPERATION_TAG": "EXT_STAGE",
        "OPERATION_TAG_VALUE": "@\"MED_PTCDH_DEV\".\"MED_PTCDH_LND\".CDH_LANDING_SAP_PAYMENTS_JP",
        "OPERATION_TAG_VALUE_GUI": null
    },
    {
        "JOB_ID": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "JOB_NAME": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "DESCRIPTION": "Job to load SAP_JP data from S3 files to landing",
        "JOB_TYPE": "snowsql",
        "JOB_CATEGORY": "data processing",
        "ACTIVE_FLAG": "Y",
        "OPERATION_ID": "4",
        "OPERATION_NAME": "execute_query",
        "OPERATION_TYPE": "S3_TO_LND_PARAMETERS",
        "OPERATION_TAG": "EXT_STAGE_ARCH",
        "OPERATION_TAG_VALUE": "@\"MED_PTCDH_DEV\".\"MED_PTCDH_LND\".CDH_ARCHIVE_SAP_PAYMENTS_JP",
        "OPERATION_TAG_VALUE_GUI": null
    },
    {
        "JOB_ID": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "JOB_NAME": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "DESCRIPTION": "Job to load SAP_JP data from S3 files to landing",
        "JOB_TYPE": "snowsql",
        "JOB_CATEGORY": "data processing",
        "ACTIVE_FLAG": "Y",
        "OPERATION_ID": "1",
        "OPERATION_NAME": "execute_query",
        "OPERATION_TYPE": "S3_TO_LND_TRUNCATE",
        "OPERATION_TAG": "query",
        "OPERATION_TAG_VALUE": "truncate table MED_PTCDH_DEV.MED_PTCDH_LND.LND_SAP_JP_TRANSACTION;",
        "OPERATION_TAG_VALUE_GUI": null
    },
    {
        "JOB_ID": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "JOB_NAME": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "DESCRIPTION": "Job to load SAP_JP data from S3 files to landing",
        "JOB_TYPE": "snowsql",
        "JOB_CATEGORY": "data processing",
        "ACTIVE_FLAG": "Y",
        "OPERATION_ID": "3",
        "OPERATION_NAME": "execute_query",
        "OPERATION_TYPE": "S3_TO_LND_PARAMETERS",
        "OPERATION_TAG": "TARGET_TABLE_NAME",
        "OPERATION_TAG_VALUE": "LND_SAP_JP_TRANSACTION",
        "OPERATION_TAG_VALUE_GUI": null
    },
    {
        "JOB_ID": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "JOB_NAME": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "DESCRIPTION": "Job to load SAP_JP data from S3 files to landing",
        "JOB_TYPE": "snowsql",
        "JOB_CATEGORY": "data processing",
        "ACTIVE_FLAG": "Y",
        "OPERATION_ID": "3",
        "OPERATION_NAME": "execute_query",
        "OPERATION_TYPE": "S3_TO_LND_PARAMETERS",
        "OPERATION_TAG": "SOURCE_FILE_NAME",
        "OPERATION_TAG_VALUE": "ZOH_HCP_TP_EXTRACT_PT_ID.DAT",
        "OPERATION_TAG_VALUE_GUI": null
    },
    {
        "JOB_ID": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "JOB_NAME": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "DESCRIPTION": "Job to load SAP_JP data from S3 files to landing",
        "JOB_TYPE": "snowsql",
        "JOB_CATEGORY": "data processing",
        "ACTIVE_FLAG": "Y",
        "OPERATION_ID": "3",
        "OPERATION_NAME": "execute_query",
        "OPERATION_TYPE": "S3_TO_LND_PARAMETERS",
        "OPERATION_TAG": "TARGET_PATH",
        "OPERATION_TAG_VALUE": "MED_PTCDH_DEV.MED_PTCDH_LND",
        "OPERATION_TAG_VALUE_GUI": null
    },
    {
        "JOB_ID": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "JOB_NAME": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "DESCRIPTION": "Job to load SAP_JP data from S3 files to landing",
        "JOB_TYPE": "snowsql",
        "JOB_CATEGORY": "data processing",
        "ACTIVE_FLAG": "Y",
        "OPERATION_ID": "3",
        "OPERATION_NAME": "execute_query",
        "OPERATION_TYPE": "S3_TO_LND_PARAMETERS",
        "OPERATION_TAG": "SOURCE_PATH",
        "OPERATION_TAG_VALUE": "s3://med-ptcdh-devamrasp20240408124911/SAP/JP/landing/ZOH_HCP_TP_EXTRACT_PT_ID.DAT",
        "OPERATION_TAG_VALUE_GUI": null
    },
    {
        "JOB_ID": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "JOB_NAME": "SAP_PAYMENT_JP_S3_TO_LANDING",
        "DESCRIPTION": "Job to load SAP_JP data from S3 files to landing",
        "JOB_TYPE": "snowsql",
        "JOB_CATEGORY": "data processing",
        "ACTIVE_FLAG": "Y",
        "OPERATION_ID": "3",
        "OPERATION_NAME": "execute_query",
        "OPERATION_TYPE": "S3_TO_LND_JOB_AUDIT_LOG",
        "OPERATION_TAG": "query",
        "OPERATION_TAG_VALUE": "CALL \"MED_PTCDH_TEST\".\"MED_PTCDH_LND\".SP_JOB_RUN_AUDIT_LOGGING('Currency_Conversion_DS_S3_TO_LANDING',3)",
        "OPERATION_TAG_VALUE_GUI": null
    }
]
