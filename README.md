import { Typography, Box, Grid, Toolbar } from '@mui/material';
import { useState, useEffect } from 'react';
import { useTranslation } from 'react-i18next';
import { useJobContext } from '../../../context/jobContext';
import SubHeader from '../../Common/SubHeader';
import { DataGrid } from '@mui/x-data-grid';
import { useNavigate, useParams } from 'react-router-dom';
import { RoutePaths } from '../../../utils/constants/routePaths';
import { getScreenSize } from '../../../utils';
import { AppContextType, useAppContext } from '../../../context/appContext';
import { getJobByID } from '../services';
import { TOAST_TYPE } from '../../../utils/enums';
import { ApiResponseType, JobContextType } from '../interface/job.interface';
import { operationConfigurationColumns } from '../constants/jobConstants';

const ViewJob = () => {
  const { id } = useParams();
  const { t } = useTranslation();
  const navigate = useNavigate();
  const jobContext: JobContextType = useJobContext();
  const screenSize = getScreenSize();
  const appContext: AppContextType = useAppContext();
  const [rows, setRows] = useState([]);

  useEffect(() => {
    appContext?.updateIsLoading(true);
    getJobByID(id)
      .then((apiResponse: ApiResponseType) => {
        mapJobData(apiResponse);
        appContext?.updateToastAttributes({
          showToast: true,
          severity: TOAST_TYPE.ERROR,
          toastMsg: t('noDataFoundError')
        });
        appContext?.updateIsLoading(false);
      })
      .catch(() => {
        appContext?.updateToastAttributes({
          showToast: true,
          severity: TOAST_TYPE.ERROR,
          toastMsg: t('apiError')
        });
        appContext?.updateIsLoading(false);
      });
    return () => {
      jobContext?.updateJobData({
        jobName: '',
        jobDescription: '',
        jobType: '',
        operationConfigRows: []
      });
    };
  }, []);

  useEffect(() => {
    console.log('jobdata', jobContext?.jobData?.operationConfigRows);
    console.log('jobdata', typeof jobContext?.jobData?.operationConfigRows);
    const formattedRows: { string: any; id: number }[] = jobContext?.jobData?.operationConfigRows?.map((row?, index?) => ({
      id: index,
      ...row
    }));
    setRows(formattedRows);
  }, [jobContext?.jobData?.operationConfigRows]);

  const mapJobData = (data?: ApiResponseType) => {
    const apiResponse = data;
    // console.log('apiResp', apiResponse?.jobDetails.JOB_ID);
    const jobName = apiResponse?.jobDetails?.JOB_NAME || '';
    const jobDescription = apiResponse?.jobDetails?.DESCRIPTION || '';
    const jobType = apiResponse?.jobDetails?.JOB_TYPE || '';

    console.log('api', apiResponse?.operationConfigurationDetails);

    let operationConfigRows: any;
    for (const key in apiResponse?.operationConfigurationDetails) {
      if (apiResponse?.operationConfigurationDetails.hasOwnProperty(key)) {
        const operation = apiResponse?.operationConfigurationDetails[key];
        operationConfigRows = {
          operationSequence: operation?.OPERATION_ID,
          operationName: operation?.OPERATION_NAME,
          operationTag: operation?.OPERATION_TAG,
          operationTagValue: operation?.OPERATION_TAG_VALUE_GUI ?? operation?.OPERATION_TAG_VALUE,
          operationType: operation?.OPERATION_TYPE
        };
        // console.log(`ID: ${operation.OPERATION_ID}`);
      }
    }
    
    console.log('rows', operationConfigRows);
    jobContext?.updateJobData({
      jobName,
      jobDescription,
      jobType,
      operationConfigRows
    });
    appContext?.updateIsLoading(false);
  };

  const handleClose = () => {
    navigate(RoutePaths?.jobRoutes?.job);
  };

  return (
    <Box component="main" sx={{ flexGrow: 1, p: screenSize?.isSmallScreen ? 2 : 5, overflowX: 'hidden' }}>
      <Toolbar />
      <SubHeader title={t('viewJob')} onClose={handleClose} compName={t('query_CompName')} btnTitle={t('edit')} closeBtnTitle={t('close')} />
      <Box className="tw-mt-5 tw-block tw-pb-2 tw-border-solid tw-border-2 tw-border-pfizerBlue" sx={{ overflowX: 'hidden' }}>
        <Typography variant="h6" component="h6" padding={1} className="tw-bg-pfizerBlue tw-text-white tw-font-bold">
          {t('jobDetailsLabel')}
        </Typography>
        <Box display="flex" flexDirection="column" padding={2} width="100%">
          <Grid container spacing={2}>
            <Grid item xs={12}>
              <Grid container spacing={2}>
                <Grid item xs={12} sm={6}>
                  <Box display="flex" alignItems="center">
                    <Typography sx={{ fontSize: 15, fontWeight: 'bold' }}>{t('query_det_name')}</Typography>
                    <Typography sx={{ marginLeft: screenSize?.isSmallScreen ? '2rem' : '5rem' }}>{jobContext?.jobData?.jobName}</Typography>
                  </Box>
                </Grid>
                <Grid item xs={12} sm={6}>
                  <Box display="flex" alignItems="center">
                    <Typography sx={{ fontSize: 15, fontWeight: 'bold' }}>{t('create_query_type')}</Typography>
                    <Typography sx={{ marginLeft: screenSize?.isSmallScreen ? '2rem' : '5rem' }}>{jobContext?.jobData?.jobType}</Typography>
                  </Box>
                </Grid>
              </Grid>
            </Grid>
            <Grid item xs={12}>
              <Grid container spacing={2}>
                <Grid item xs={12} sm={6}>
                  <Box display="flex" alignItems="center">
                    <Typography sx={{ fontSize: 15, fontWeight: 'bold' }}>{t('query_description')}</Typography>
                    <Typography sx={{ marginLeft: screenSize?.isSmallScreen ? '2rem' : '2.75rem' }}>{jobContext?.jobData?.jobDescription}</Typography>
                  </Box>
                </Grid>
              </Grid>
            </Grid>
          </Grid>
        </Box>
      </Box>
      <Box className="tw-mt-5 tw-block tw-pb-2 tw-border-solid tw-border-2 tw-border-pfizerBlue" sx={{ overflowX: 'hidden' }}>
        <Typography variant="h6" component="h6" padding={1} className="tw-bg-pfizerBlue tw-text-white tw-font-bold">
          {t('jobOperationConfigLabel')}
        </Typography>
        <Box className="tw-w-[100%] tw-flex tw-justify-center tw-items-center tw-p-5" sx={{ overflowX: 'auto' }}>
          <DataGrid
            autoHeight
            rows={rows}
            columns={operationConfigurationColumns}
            pageSizeOptions={[10]}
            disableRowSelectionOnClick
            disableColumnFilter
            disableColumnSelector
            disableDensitySelector
            disableColumnMenu
            disableColumnResize
            rowHeight={70}
            className="tw-min-w-[30vw]"
            sx={{
              '--DataGrid-overlayHeight': '50vh',
              '& .custom-header': { backgroundColor: '#F2F4F8', color: 'black' },
              '& .custom-cell': {
                backgroundColor: 'white',
                padding: '0.625rem',
                display: 'flex',
                alignItems: 'center',
                justifyContent: 'left',
                minHeight: '5vh'
              },
              '& .MuiDataGrid-cell:focus-within': {
                outline: 'none !important'
              }
            }}
          />
        </Box>
      </Box>
    </Box>
  );
};

export default ViewJob;


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
}
