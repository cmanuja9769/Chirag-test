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
    const formattedRows = jobContext?.jobData?.operationConfigRows?.map((row, index) => ({
      id: index,
      ...row
    }));
    setRows(formattedRows);
  }, [jobContext?.jobData?.operationConfigRows]);

  const mapJobData = (data: ApiResponseType) => {
    const jobDetails = data?.jobDetails;
    const jobName = jobDetails?.JOB_NAME || '';
    const jobDescription = jobDetails?.DESCRIPTION || '';
    const jobType = jobDetails?.JOB_TYPE || '';

    const operationConfigRows = data?.operationConfigurationDetails?.map((operation) => ({
      operationSequence: operation?.OPERATION_ID,
      operationName: operation?.OPERATION_NAME,
      operationTag: operation?.OPERATION_TAG,
      operationTagValue: operation?.OPERATION_TAG_VALUE_GUI ?? operation?.OPERATION_TAG_VALUE,
      operationType: operation?.OPERATION_TYPE
    })) || [];

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