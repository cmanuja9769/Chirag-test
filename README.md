APIs:

localhost:3001/api/v1/criteria?identifier=IDENTIFIER_PTCDH_SNOWFLAKE_CONNECTIONS_WITH_QUERYSTRING&search=Med
method: GET

returns something like this : [
    {
        "CONNECTION_NAME": "GEN-AI",
        "CONNECTION_ID": "9927344209"
    }
]

localhost:3001/api/v1/schema/getSchemaByConnection?connectionId=7599058705
method: GET

returns something like this : {
    "schema": [
        {
            "ENTITY_NAME": "PTCDH Landing Layer",
            "SCHEMA_NAME": "PTCDH_LND"
        }
    ]
}

localhost:3001/api/v1/schema/getSchemaByConnection?connectionId=7599058705&schemaName=PTCDH_LND
method: GET

 returns something like this : {
    "objects": [
        "LND_SAP_US_TRANSACTION",
        "LND_SAP_JP_TRANSACTION",
        "LND_SAP_GLBL2_TRANSACTION",
        "LND_SAP_GLBL1_TRANSACTION",
        "LND_SAP_GLBL_TRANSACTION",
        "LND_SAMPLES_TRANSACTION"
    ]
}

import { useTranslation } from 'react-i18next';
import React, { useEffect, useRef, useState } from 'react';
import { Box, FormControl, Select, MenuItem, Typography, FormHelperText, Grid } from '@mui/material';

import { Field, Form, Formik } from 'formik';
import { useJobContext } from '../../../../context/jobContext';

import * as Yup from 'yup';
import TooltipComponent from '../../../Common/Tooltip';
import { TOAST_TYPE } from '../../../../utils/enums';
import ToastMessage from '../../../Common/ToastMessage/ToastMessage';

const JobTargetConfig: React.FC = () => {
  const { t } = useTranslation();
  const { jobData, updateJobData, setJobTargetValidation, isNexButtonClicked } = useJobContext();
  const formikRef = useRef<any>(null);
  const [openToast, setOpenToast] = useState<boolean>(false);
  const [toastInfo, setToastInfo] = useState<{ severity: TOAST_TYPE; message: string }>({ message: '', severity: TOAST_TYPE.INFO });

  const validationSchema = Yup.object({
    target_conName: Yup.string().required(t('create_job_conName_reqd')),
    target_schema: Yup.string().required(t('create_job_schema_reqd')),
    target_object: Yup.string().required(t('create_job_object_reqd'))
  });

  const handleChange = (e?: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e?.target;
    updateJobData((prevJobData?) => ({
      ...prevJobData,
      [name]: value
    }));
  };

  const handleToast = (message, type) => {
    setOpenToast(true);
    setToastInfo({
      message: message,
      severity: type
    });
  };

  const validForm = async () => {
    if (formikRef?.current) {
      try {
        await formikRef?.current?.submitForm();
      } catch (error) {
        handleToast(t('form_error'), TOAST_TYPE.ERROR);
      }
    }
  };

  useEffect(() => {
    setJobTargetValidation(() => async () => {
      try {
        await validationSchema?.validate(jobData, { abortEarly: false });
        return Promise.resolve();
      } catch (errors) {
        return Promise.reject(errors);
      }
    });
  }, [jobData, setJobTargetValidation]);

  useEffect(() => {
    if (isNexButtonClicked) {
      validForm();
    }
  }, [isNexButtonClicked]);

  return (
    <Box className="tw-mt-5 tw-block tw-pb-2 tw-border-solid tw-border-2 tw-border-pfizerBlue">
      <Typography variant="h6" component="h6" padding={1} className="tw-bg-pfizerBlue tw-text-white tw-font-bold">
        {t('job_target_label')}
      </Typography>
      <Formik
        innerRef={formikRef}
        initialValues={jobData}
        validationSchema={validationSchema}
        validateOnChange={false}
        validateOnBlur={true}
        onSubmit={(values, { setSubmitting }) => {
          updateJobData(values);
          setSubmitting(false);
        }}
      >
        {({ values, handleBlur, errors, touched, setFieldValue }) => (
          <Form>
            <Box display="inline-flex" alignItems="center" padding={2}>
              <Grid container spacing={2} direction="row" alignItems="center" justifyContent="center">
                <Grid item xs={12} sm={2} margin="auto" display="inline-flex">
                  <Typography sx={{ fontSize: 15, fontWeight: 'bold' }}>{t('job_target_conName')}</Typography>
                  <TooltipComponent title={t('job_target_conName')} />
                </Grid>
                <Grid item xs={12} sm={10}>
                  <FormControl margin="normal" error={touched?.target_conName && Boolean(errors?.target_conName)}>
                    <Field
                      sx={{ width: '20vw' }}
                      as={Select}
                      name="target_conName"
                      value={values?.target_conName}
                      onChange={(e: React.ChangeEvent<HTMLInputElement>) => {
                        setFieldValue('target_conName', e?.target?.value);
                        handleChange(e);
                      }}
                      onBlur={handleBlur}
                    >
                      <MenuItem value={t('job_det_status_snowflake')}>{t('job_det_status_snowflake')}</MenuItem>
                      <MenuItem value={t('job_det_status_databrick')}>{t('job_det_status_databrick')}</MenuItem>
                      <MenuItem value={t('job_det_status_trans')}>{t('job_det_status_trans')}</MenuItem>
                    </Field>
                    {touched?.target_conName && errors?.target_conName && (
                      <FormHelperText className="tw-absolute tw-bottom-[-1.2vw]">{errors?.target_conName}</FormHelperText>
                    )}
                  </FormControl>
                </Grid>
                <Grid item xs={12} sm={12} display="inline-flex" alignItems="center" padding={2}>
                  <Grid item xs={12} sm={2} marginTop="3vh" display="inline-flex">
                    <Typography sx={{ fontSize: 15, fontWeight: 'bold' }}>{t('job_target_schema')}</Typography>
                    <TooltipComponent title={t('job_target_schema')} />
                  </Grid>
                  <Grid item xs={12} sm={4}>
                    <FormControl margin="normal" error={touched?.target_schema && Boolean(errors?.target_schema)}>
                      <Field
                        sx={{ width: '20vw' }}
                        as={Select}
                        name="target_schema"
                        value={values?.target_schema}
                        onChange={(e: React.ChangeEvent<HTMLInputElement>) => {
                          setFieldValue('target_schema', e?.target?.value);
                          handleChange(e);
                        }}
                        onBlur={handleBlur}
                      >
                        <MenuItem value={t('job_det_status_snowflake')}>{t('job_det_status_snowflake')}</MenuItem>
                        <MenuItem value={t('job_det_status_databrick')}>{t('job_det_status_databrick')}</MenuItem>
                        <MenuItem value={t('job_det_status_trans')}>{t('job_det_status_trans')}</MenuItem>
                      </Field>
                      {touched?.target_schema && errors?.target_schema && (
                        <FormHelperText className="tw-absolute tw-bottom-[-1.2vw]">{errors?.target_schema}</FormHelperText>
                      )}
                    </FormControl>
                  </Grid>
                  <Grid item xs={12} sm={2} marginTop="3vh" display="inline-flex">
                    <Typography sx={{ fontSize: 15, fontWeight: 'bold' }}>{t('job_target_object')}</Typography>
                    <TooltipComponent title={t('job_target_object')} />
                  </Grid>
                  <Grid item xs={12} sm={4}>
                    <FormControl margin="normal" error={touched?.target_object && Boolean(errors?.target_object)}>
                      <Field
                        sx={{ width: '20vw' }}
                        as={Select}
                        name="target_object"
                        value={values?.target_object}
                        onChange={(e: React.ChangeEvent<HTMLInputElement>) => {
                          setFieldValue('target_object', e?.target?.value);
                          handleChange(e);
                        }}
                        onBlur={handleBlur}
                      >
                        <MenuItem value={t('job_det_status_snowflake')}>{t('job_det_status_snowflake')}</MenuItem>
                        <MenuItem value={t('job_det_status_databrick')}>{t('job_det_status_databrick')}</MenuItem>
                        <MenuItem value={t('job_det_status_trans')}>{t('job_det_status_trans')}</MenuItem>
                      </Field>
                      {touched?.target_object && errors?.target_object && (
                        <FormHelperText className="tw-absolute tw-bottom-[-1.2vw]">{errors?.target_object}</FormHelperText>
                      )}
                    </FormControl>
                  </Grid>
                </Grid>
              </Grid>
            </Box>
          </Form>
        )}
      </Formik>
      <ToastMessage severity={toastInfo.severity} isVisible={openToast} hideToast={() => setOpenToast(false)} message={toastInfo.message} />
    </Box>
  );
};

export default JobTargetConfig;



jobContext.tsx

import { createContext, useContext, useState, ReactNode } from 'react';
import { JobContextProps, JobProviderProps } from '../components/Jobs/job.interface';

const JobContext = createContext<JobContextProps>({
  jobData: null,
  step: 0,
  headers: [],
  charCount: 0,
  updateJobData: () => {},
  updateStage: () => {},
  updateHeaders: () => {},
  setCharCount: () => {},
  validateJobDetails: () => Promise.resolve(),
  validateJobSource: () => Promise.resolve(),
  validateJobTarget: () => Promise.resolve(),
  setJobDetailsValidation: () => {},
  setJobSourceValidation: () => {},
  setJobTargetValidation: () => {},
  isNexButtonClicked: false,
  handleNextClick: () => {}
});

export const JobProvider = ({ children }: JobProviderProps) => {
  const [jobData, setJobData] = useState({
    job_name: '',
    description: '',
    job_type: '',
    source_conName: '',
    source_schema: '',
    source_object: '',
    target_conName: '',
    target_object: '',
    target_schema: '',
    selectedOption: '',
    checkboxStates: {},
    previewData: {}
  });

  const [headers, setHeaders] = useState([]);
  const [step, setStep] = useState(0);
  const [charCount, setCharCount] = useState(256);
  const [validateJobDetails, setJobDetailsValidation] = useState(() => () => Promise.resolve());
  const [validateJobSource, setJobSourceValidation] = useState(() => () => Promise.resolve());
  const [validateJobTarget, setJobTargetValidation] = useState(() => () => Promise.resolve());
  const [isNexButtonClicked, setNextButtonClicked] = useState(false);

  const updateJobData = (value?: any) => {
    setJobData(value);
  };

  const updateStage = (value?: any) => {
    setStep(value);
  };

  const updateHeaders = (value?: any) => {
    setHeaders(value);
  };

  const handleNextClick = (value?: boolean) => {
    setNextButtonClicked(value);
  };

  const updateCheckboxState = (checkbox?: string) => {
    setJobData((prevState?) => ({
      ...prevState,
      checkboxStates: {
        ...prevState?.checkboxStates,
        [checkbox]: !prevState.checkboxStates[checkbox]
      }
    }));
  };

  const updateSelectedOption = (option?: string) => {
    setJobData((prevState) => ({
      ...prevState,
      selectedOption: option,
      checkboxStates: {}
    }));
  };

  const updatePreviewData = (checkbox?: string, data?: string) => {
    setJobData((prevState?) => ({
      ...prevState,
      previewData: {
        ...prevState?.previewData,
        [checkbox]: data
      }
    }));
  };

  const context: JobContextProps = {
    jobData,
    step,
    headers,
    charCount,
    updateJobData,
    updateStage,
    updateHeaders,
    setCharCount,
    validateJobDetails,
    validateJobSource,
    validateJobTarget,
    setJobDetailsValidation,
    setJobSourceValidation,
    setJobTargetValidation,
    isNexButtonClicked,
    handleNextClick,
    updateCheckboxState,
    updateSelectedOption,
    updatePreviewData
  };

  return <JobContext.Provider value={context}>{children}</JobContext.Provider>;
};

export const useJobContext = () => useContext(JobContext);
