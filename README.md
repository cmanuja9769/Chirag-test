import { useTranslation } from 'react-i18next';
import React, { useEffect, useRef, useState } from 'react';
import { Box, FormControl, Autocomplete, TextField, Typography, FormHelperText, Grid } from '@mui/material';

import { Field, Form, Formik } from 'formik';
import { useJobContext } from '../../../../context/jobContext';
import axios from 'axios';
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

  const [connectionOptions, setConnectionOptions] = useState<any[]>([]);
  const [schemaOptions, setSchemaOptions] = useState<any[]>([]);
  const [objectOptions, setObjectOptions] = useState<string[]>([]);

  const validationSchema = Yup.object({
    target_conName: Yup.string().required(t('create_job_conName_reqd')),
    target_schema: Yup.string().required(t('create_job_schema_reqd')),
    target_object: Yup.string().required(t('create_job_object_reqd'))
  });

  const handleChange = (name: string, value: any) => {
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

  const fetchConnections = async (searchValue: string) => {
    try {
      const response = await axios.get(`http://localhost:3001/api/v1/criteria?identifier=IDENTIFIER_PTCDH_SNOWFLAKE_CONNECTIONS_WITH_QUERYSTRING&search=${searchValue}`);
      setConnectionOptions(response.data);
    } catch (error) {
      handleToast(t('fetch_error'), TOAST_TYPE.ERROR);
    }
  };

  const fetchSchemas = async (connectionId: string) => {
    try {
      const response = await axios.get(`http://localhost:3001/api/v1/schema/getSchemaByConnection?connectionId=${connectionId}`);
      setSchemaOptions(response.data.schema);
    } catch (error) {
      handleToast(t('fetch_error'), TOAST_TYPE.ERROR);
    }
  };

  const fetchObjects = async (connectionId: string, schemaName: string) => {
    try {
      const response = await axios.get(`http://localhost:3001/api/v1/schema/getSchemaByConnection?connectionId=${connectionId}&schemaName=${schemaName}`);
      setObjectOptions(response.data.objects);
    } catch (error) {
      handleToast(t('fetch_error'), TOAST_TYPE.ERROR);
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
                    <Autocomplete
                      sx={{ width: '20vw' }}
                      options={connectionOptions.map(option => option.CONNECTION_NAME)}
                      value={values?.target_conName}
                      onInputChange={(e, newValue) => {
                        fetchConnections(newValue);
                        setFieldValue('target_conName', newValue);
                        handleChange('target_conName', newValue);
                      }}
                      onChange={(e, newValue) => {
                        const selectedConnection = connectionOptions.find(option => option.CONNECTION_NAME === newValue);
                        if (selectedConnection) {
                          fetchSchemas(selectedConnection.CONNECTION_ID);
                          setFieldValue('target_conName', newValue);
                          handleChange('target_conName', newValue);
                        }
                      }}
                      renderInput={(params) => <TextField {...params} label={t('job_target_conName')} />}
                    />
                    {touched?.target_conName && errors?.target_conName && (
                      <FormHelperText className="tw-absolute tw-bottom-[-1.2vw]">{errors?.target_conName}</FormHelperText>
                    )}
                  </FormControl>
                </Grid>
                <Grid item xs={12} sm={2} marginTop="3vh" display="inline-flex">
                  <Typography sx={{ fontSize: 15, fontWeight: 'bold' }}>{t('job_target_schema')}</Typography>
                  <TooltipComponent title={t('job_target_schema')} />
                </Grid>
                <Grid item xs={12} sm={4}>
                  <FormControl margin="normal" error={touched?.target_schema && Boolean(errors?.target_schema)}>
                    <Autocomplete
                      sx={{ width: '20vw' }}
                      options={schemaOptions.map(option => option.SCHEMA_NAME)}
                      value={values?.target_schema}
                      onChange={(e, newValue) => {
                        const selectedSchema = schemaOptions.find(option => option.SCHEMA_NAME === newValue);
                        if (selectedSchema) {
                          fetchObjects(values?.target_conName, newValue);
                          setFieldValue('target_schema', newValue);
                          handleChange('target_schema', newValue);
                        }
                      }}
                      renderInput={(params) => <TextField {...params} label={t('job_target_schema')} />}
                    />
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
                    <Autocomplete
                      sx={{ width: '20vw' }}
                      options={objectOptions}
                      value={values?.target_object}
                      onChange={(e, newValue) => {
                        setFieldValue('target_object', newValue);
                        handleChange('target_object', newValue);
                      }}
                      renderInput={(params) => <TextField {...params} label={t('job_target_object')} />}
                    />
                    {touched?.target_object && errors?.target_object && (
                      <FormHelperText className="tw-absolute tw-bottom-[-1.2vw]">{errors?.target_object}</FormHelperText>
                    )}
                  </FormControl>
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



import React, { createContext, useContext, useState, ReactNode, useCallback } from 'react';

interface JobData {
  job_name: string;
  description: string;
  job_type: string;
  source_conName: string;
  source_schema: string;
  source_object: string;
  target_conName: string;
  target_schema: string;
  target_object: string;
  selectedOption: string;
  checkboxStates: Record<string, boolean>;
  previewData: Record<string, string>;
}

interface JobContextProps {
  jobData: JobData;
  step: number;
  headers: any[];
  charCount: number;
  updateJobData: (newData: Partial<JobData>) => void;
  updateStage: (value: number) => void;
  updateHeaders: (headers: any[]) => void;
  setCharCount: (count: number) => void;
  validateJobDetails: () => Promise<void>;
  validateJobSource: () => Promise<void>;
  validateJobTarget: () => Promise<void>;
  setJobDetailsValidation: (validationFn: () => Promise<void>) => void;
  setJobSourceValidation: (validationFn: () => Promise<void>) => void;
  setJobTargetValidation: (validationFn: () => Promise<void>) => void;
  isNexButtonClicked: boolean;
  handleNextClick: (clicked: boolean) => void;
}

interface JobProviderProps {
  children: ReactNode;
}

const JobContext = createContext<JobContextProps | undefined>(undefined);

export const JobProvider = ({ children }: JobProviderProps) => {
  const [jobData, setJobData] = useState<JobData>({
    job_name: '',
    description: '',
    job_type: '',
    source_conName: '',
    source_schema: '',
    source_object: '',
    target_conName: '',
    target_schema: '',
    target_object: '',
    selectedOption: '',
    checkboxStates: {},
    previewData: {}
  });

  const [headers, setHeaders] = useState<any[]>([]);
  const [step, setStep] = useState(0);
  const [charCount, setCharCount] = useState(256);
  const [validateJobDetails, setJobDetailsValidation] = useState<() => Promise<void>>(() => async () => {});
  const [validateJobSource, setJobSourceValidation] = useState<() => Promise<void>>(() => async () => {});
  const [validateJobTarget, setJobTargetValidation] = useState<() => Promise<void>>(() => async () => {});
  const [isNexButtonClicked, setNextButtonClicked] = useState(false);

  const updateJobData = useCallback((newData: Partial<JobData>) => {
    setJobData((prevData) => ({
      ...prevData,
      ...newData
    }));
  }, []);

  const contextValue = {
    jobData,
    step,
    headers,
    charCount,
    updateJobData,
    updateStage: setStep,
    updateHeaders: setHeaders,
    setCharCount,
    validateJobDetails,
    validateJobSource,
    validateJobTarget,
    setJobDetailsValidation,
    setJobSourceValidation,
    setJobTargetValidation,
    isNexButtonClicked,
    handleNextClick: setNextButtonClicked
  };

  return <JobContext.Provider value={contextValue}>{children}</JobContext.Provider>;
};

export const useJobContext = (): JobContextProps => {
  const context = useContext(JobContext);
  if (!context) {
    throw new Error('useJobContext must be used within a JobProvider');
  }
  return context;
};
