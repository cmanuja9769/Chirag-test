import { useTranslation } from 'react-i18next';
import React, { useEffect, useRef, useState } from 'react';
import { Box, FormControl, Autocomplete, TextField, Typography, FormHelperText, Grid } from '@mui/material';

import { Form, Formik } from 'formik';
import { useJobContext } from '../../../../context/jobContext';
import axios from 'axios';
import * as Yup from 'yup';
import TooltipComponent from '../../../Common/Tooltip';
import { TOAST_TYPE } from '../../../../utils/enums';
import ToastMessage from '../../../Common/ToastMessage/ToastMessage';
import apiConfig from '../../../../utils/apiConfig';
import { JobEndpoints } from '../../../../utils/constants/apiEndpoints';
import { useFetcher } from '../../../../hooks/useReactQuery';

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

  const dataFormation = (results: any) => {
    console.log('recd');
    if (results) {
      console.log(results);
    }
  };

  const fetchRequest = (url?: string) => {
    console.log('Hello', url);
    const { data, isLoading, isFetching, refetch, isError } = useFetcher({
      url: url,
      queryKey: [],
      enableQuery: true,
      dataFormatter: dataFormation
    });
  };

  const handleChange = (name?: string, value?: any) => {
    updateJobData((prevJobData?) => ({
      ...prevJobData,
      [name]: value
    }));
  };

  const handleToast = (message?, type?) => {
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
      const url = `${apiConfig?.apiURL}${JobEndpoints?.searchConnections}&search=${searchValue}`;
      console.log('url', url);
      const conNameObj = 'target_conName';
      fetchRequest(url);
      // setConnectionOptions(response?.data);
    } catch (error) {
      handleToast(t('fetch_error'), TOAST_TYPE.ERROR);
    }
  };

  const fetchSchemas = async (connectionId?: string) => {
    try {
      const response = await axios?.get(`${apiConfig?.apiURL}${JobEndpoints?.getSchemaByConnection}?connectionId=${connectionId}`);
      setSchemaOptions(response?.data?.schema);
    } catch (error) {
      handleToast(t('fetch_error'), TOAST_TYPE.ERROR);
    }
  };

  const fetchObjects = async (connectionId?: string, schemaName?: string) => {
    try {
      const response = await axios?.get(
        `${apiConfig.apiURL}${JobEndpoints?.getSchemaByConnection}?connectionId=${connectionId}&schemaName=${schemaName}`
      );
      setObjectOptions(response?.data?.objects);
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
                      options={connectionOptions?.map((option?) => option?.CONNECTION_NAME)}
                      value={values?.target_conName}
                      onInputChange={(e, newValue) => {
                        fetchConnections(newValue);
                        setFieldValue(t('target_conName'), newValue);
                        handleChange(t('target_conName'), newValue);
                        setFieldValue(t('target_schema'), '');
                        setFieldValue(t('target_object'), '');
                        handleChange(t('target_schema'), '');
                        handleChange(t('target_object'), '');
                        setSchemaOptions([]);
                        setObjectOptions([]);
                      }}
                      onChange={(e, newValue) => {
                        const selectedConnection = connectionOptions?.find((option?) => option?.CONNECTION_NAME === newValue);
                        if (selectedConnection) {
                          fetchSchemas(selectedConnection?.CONNECTION_ID);
                          setFieldValue(t('target_conName'), newValue);
                          handleChange(t('target_conName'), newValue);
                        }
                        setFieldValue(t('target_schema'), '');
                        setFieldValue(t('target_object'), '');
                        handleChange(t('target_schema'), '');
                        handleChange(t('target_object'), '');
                        setSchemaOptions([]);
                        setObjectOptions([]);
                      }}
                      renderInput={(params?) => <TextField {...params} />}
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
                      options={schemaOptions?.map((option?) => option?.SCHEMA_NAME)}
                      value={values?.target_schema}
                      onChange={(e, newValue) => {
                        const selectedConnection = connectionOptions?.find((option?) => option?.CONNECTION_NAME === values?.target_conName);
                        const selectedSchema = schemaOptions?.find((option?) => option?.SCHEMA_NAME === newValue);
                        if (selectedSchema) {
                          fetchObjects(selectedConnection?.CONNECTION_ID, newValue);
                          setFieldValue(t('target_schema'), newValue);
                          handleChange(t('target_schema'), newValue);
                        }
                        setFieldValue(t('target_object'), '');
                        handleChange(t('target_object'), '');
                        setObjectOptions([]);
                      }}
                      renderInput={(params?) => <TextField {...params} />}
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
                        setFieldValue(t('target_object'), newValue);
                        handleChange(t('target_object'), newValue);
                      }}
                      renderInput={(params?) => <TextField {...params} />}
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
