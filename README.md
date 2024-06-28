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

const TargetConfiguration: React.FC = () => {
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

  const fetchConnections = async (searchValue?: string) => {
    try {
      const response = await axios.get(
        `http://localhost:3001/api/v1/criteria?identifier=IDENTIFIER_PTCDH_SNOWFLAKE_CONNECTIONS_WITH_QUERYSTRING&search=${searchValue}`
      );
      setConnectionOptions(response?.data);
    } catch (error) {
      handleToast(t('fetch_error'), TOAST_TYPE.ERROR);
    }
  };

  const fetchSchemas = async (connectionId?: string) => {
    try {
      const response = await axios.get(`http://localhost:3001/api/v1/schema/getSchemaByConnection?connectionId=${connectionId}`);
      setSchemaOptions(response?.data?.schema);
    } catch (error) {
      handleToast(t('fetch_error'), TOAST_TYPE.ERROR);
    }
  };

  const fetchObjects = async (connectionId?: string, schemaName?: string) => {
    try {
      const response = await axios?.get(
        `http://localhost:3001/api/v1/schema/getSchemaByConnection?connectionId=${connectionId}&schemaName=${schemaName}`
      );
      setObjectOptions(response?.data.objects);
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
                      options={connectionOptions.map((option) => option.CONNECTION_NAME)}
                      value={values?.target_conName}
                      onInputChange={(e, newValue) => {
                        fetchConnections(newValue);
                        setFieldValue('target_conName', newValue);
                        handleChange('target_conName', newValue);
                      }}
                      onChange={(e, newValue) => {
                        const selectedConnection = connectionOptions.find((option) => option.CONNECTION_NAME === newValue);
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
                      options={schemaOptions.map((option) => option.SCHEMA_NAME)}
                      value={values?.target_schema}
                      onChange={(e, newValue) => {
                        const selectedConnection = connectionOptions.find((option) => option.CONNECTION_NAME === values?.target_conName);
                        const selectedSchema = schemaOptions.find((option) => option.SCHEMA_NAME === newValue);
                        if (selectedSchema) {
                          fetchObjects(selectedConnection.CONNECTION_ID, newValue);
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

export default TargetConfiguration;



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


import { ReactNode } from 'react';

export interface JobDataProps {
  id?: number;
  name?: string;
  tags?: string[];
  createdtime?: string;
  description?: string;
  status?: string;
}

export interface JobData {
  job_name?: string;
  description?: string;
  job_type?: string;
  source_conName?: string;
  source_schema?: string;
  source_object?: string;
  target_conName?: string;
  target_object?: string;
  target_schema?: string;
  selectedOption?: string;
  checkboxStates?: { [key: string]: boolean };
  previewData?: { [key: string]: string };
}

export interface JobContextProps {
  jobData?: JobData;
  step?: number;
  headers: any[];
  charCount: number;
  updateJobData?: React.Dispatch<React.SetStateAction<JobData>>;
  updateStage?: React.Dispatch<React.SetStateAction<number>>;
  setValidateJobDetailsForm?: React.Dispatch<React.SetStateAction<() => Promise<void>>>;
  setValidateJobParamsForm?: React.Dispatch<React.SetStateAction<() => Promise<void>>>;
  updateHeaders?: React.Dispatch<React.SetStateAction<any[]>>;
  setCharCount: React.Dispatch<React.SetStateAction<number>>;
  validateJobDetails?: () => Promise<void>;
  validateJobSource?: () => Promise<void>;
  validateJobTarget?: () => Promise<void>;
  setJobDetailsValidation?: React.Dispatch<React.SetStateAction<() => Promise<void>>>;
  setJobSourceValidation?: React.Dispatch<React.SetStateAction<() => Promise<void>>>;
  setJobTargetValidation?: React.Dispatch<React.SetStateAction<() => Promise<void>>>;
  isNexButtonClicked?: boolean;
  handleNextClick?: React.Dispatch<React.SetStateAction<boolean>>;
  updateCheckboxState?: (checkbox: string) => void;
  updateSelectedOption?: (option: string) => void;
  updatePreviewData?: (checkbox: string, data: string) => void;
}

export interface JobProviderProps {
  children?: ReactNode;
}

export interface JobHeaderProps {
  onClose?: () => void;
  step?: number;
  onBack?: () => void;
}
