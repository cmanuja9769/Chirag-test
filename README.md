import { useTranslation } from 'react-i18next';
import React, { useEffect, useRef, useState } from 'react';
import { Box, TextField, FormControl, Select, MenuItem, Typography, FormHelperText, Grid, Button } from '@mui/material';

import { Field, Form, Formik } from 'formik';
import { useJobContext } from '../../../../context/jobContext';

import * as Yup from 'yup';

import themeFile from '../../../../styles/theme.json';

interface MyFormProps {
  formRef: React.RefObject<Formik>;
}

const JobDetails: React.FC<MyFormProps> = ({ formRef }) => {
  const { t } = useTranslation();
  const { jobData, updateJobData, setJobDetailsValidation, isNexButtonClicked, handleNextClick } = useJobContext();
  const [charCount, setCharCount] = useState(jobData?.description?.length || 0);
  const formikRef = useRef<Formik>(null);

  const validationSchema = Yup.object({
    job_name: Yup.string().min(2, 'Too Short!').max(50, 'Too Long!').required(t('dag_name_reqd')),
    description: Yup.string().min(2, 'Too Short!').max(256, t('no_chars')).required(t('dag_description_reqd')),
    job_type: Yup.string().required(t('create_dag_status_reqd'))
  });

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    updateJobData((prevJobData) => ({
      ...prevJobData,
      [name]: value
    }));
  };

  useEffect(() => {
    const validForm = async () => {
      if (formikRef.current) {
        try {
          await formikRef.current.submitForm();
          const errors = await formikRef.current.validateForm();
          if (Object.keys(errors).length === 0) {
            setJobDetailsValidation(() => Promise.resolve());
            console.log('No errors from job details', errors);
          } else {
            console.log('Errors from job details', errors);
          }
        } catch (error) {
          setJobDetailsValidation(() => Promise.reject());
          console.error('Error during validation:', error);
        }
      }
    };

    if (isNexButtonClicked) {
      validForm();
    }
  }, [isNexButtonClicked, setJobDetailsValidation]);

  return (
    <Box
      sx={{
        border: `2px solid ${themeFile.colors.pfizerBlue}`,
        mt: 5,
        display: 'block',
        paddingBottom: 2
      }}
    >
      <Typography
        variant="h6"
        component="h6"
        padding={1}
        style={{ background: themeFile.colors.pfizerBlue, color: themeFile.colors.white, fontWeight: 'bold' }}
      >
        {t('job_det_label')}
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
          console.log('Submitted values:', values);
        }}
      >
        {({ values, handleBlur, errors, touched, setFieldValue }) => (
          <Form>
            <Box display="inline-flex" alignItems="center" padding={2}>
              <Grid container spacing={2} direction="row" alignItems="center" justifyContent="center">
                <Grid item xs={12} sm={12} display="inline-flex" justifyContent="space-around" alignItems="center">
                  <Grid item xs={12} sm={2} marginTop="1vh" display="inline-flex">
                    <Typography sx={{ fontSize: 15, fontWeight: 'bold' }}>{t('job_det_name')}</Typography>
                  </Grid>
                  <Grid item xs={12} sm={5}>
                    <Field
                      sx={{ width: '20vw' }}
                      as={TextField}
                      placeholder="Job Name"
                      name="job_name"
                      size="small"
                      value={values.job_name}
                      onChange={(e: React.ChangeEvent<HTMLInputElement>) => {
                        setFieldValue('job_name', e.target.value);
                        handleChange(e);
                      }}
                      onBlur={handleBlur}
                      margin="normal"
                      error={touched.job_name && Boolean(errors.job_name)}
                      helperText={touched.job_name && errors.job_name}
                    />
                  </Grid>
                  <Grid item xs={12} sm={1} marginTop="1vh" display="inline-flex">
                    <Typography sx={{ fontSize: 15, fontWeight: 'bold' }}>{t('create_job_type')}</Typography>
                  </Grid>
                  <Grid item xs={12} sm={4}>
                    <FormControl margin="normal" error={touched.job_type && Boolean(errors.job_type)}>
                      <Field
                        sx={{ width: '20vw' }}
                        as={Select}
                        name="job_type"
                        value={values.job_type}
                        onChange={(e: React.ChangeEvent<{ value: unknown }>) => {
                          setFieldValue('job_type', e.target.value as string);
                          handleChange(e as React.ChangeEvent<HTMLInputElement>);
                        }}
                        onBlur={handleBlur}
                      >
                        <MenuItem value={t('job_det_status_snowflake')}>{t('job_det_status_snowflake')}</MenuItem>
                        <MenuItem value={t('job_det_status_databrick')}>{t('job_det_status_databrick')}</MenuItem>
                        <MenuItem value={t('job_det_status_trans')}>{t('job_det_status_trans')}</MenuItem>
                      </Field>
                      {touched.job_type && errors.job_type && (
                        <FormHelperText>{errors.job_type}</FormHelperText>
                      )}
                    </FormControl>
                  </Grid>
                </Grid>

                <Grid item xs={12} sm={2} margin="auto" display="inline-flex">
                  <Typography sx={{ fontSize: 15, fontWeight: 'bold' }}>{t('job_description')}</Typography>
                </Grid>
                <Grid item xs={12} sm={10}>
                  <Field
                    sx={{ width: '25vw' }}
                    as={TextField}
                    placeholder="Description"
                    name="description"
                    multiline
                    rows={4}
                    value={values.description}
                    onChange={(e: React.ChangeEvent<HTMLInputElement>) => {
                      if (e.target.value.length <= 256) {
                        setFieldValue('description', e.target.value);
                        setCharCount(e.target.value.length);
                        handleChange(e);
                      }
                    }}
                    onBlur={handleBlur}
                    margin="normal"
                    error={touched.description && Boolean(errors.description)}
                    helperText={(touched.description && errors.description) || `${256 - charCount} characters remaining`}
                  />
                </Grid>
              </Grid>
            </Box>
          </Form>
        )}
      </Formik>
    </Box>
  );
};

export default JobDetails;


import React from 'react';
import { Box, Button, Typography } from '@mui/material';
import { useTranslation } from 'react-i18next';
import themeFile from '../../../styles/theme.json';
import { HeaderProps } from './header.interface';
import { useJobContext } from '../../../context/jobContext';
import { useValidation } from '../../../utils/validation';

const HeaderComponent: React.FC<HeaderProps> = ({
  title,
  onClose,
  onBack,
  step,
  validateDetails,
  validateParams,
  validateTargetParams,
  compName
}) => {
  const { t } = useTranslation();
  const jobContext = useJobContext();
  const { triggerValidation } = useValidation(compName, validateDetails, validateParams, validateTargetParams);

  const handleNext = async () => {
    try {
      jobContext.handleNextClick(true);
      await triggerValidation();
      jobContext.updateStage(step + 1);
    } catch (error) {
      console.error('Validation error:', error);
      // Handle error gracefully, e.g., display an error message in UI
    }
  };

  return (
    <Box component="header" display="flex" justifyContent="space-between" padding={1}>
      <Typography variant="h5" component="h5" paddingTop={1} style={{ color: themeFile.colors.pfizerBlue, fontWeight: 'bold' }}>
        {title}
      </Typography>
      <Box>
        {step > 0 && (
          <Button variant="outlined" color="primary" onClick={onBack} style={{ marginRight: '8px' }}>
            {t('back')}
          </Button>
        )}
        <Button variant="contained" color="primary" onClick={handleNext}>
          {t('next')}
        </Button>
        <Button variant="outlined" color="secondary" onClick={onClose} style={{ marginLeft: '8px' }}>
          {t('close')}
        </Button>
      </Box>
    </Box>
  );
};

export default HeaderComponent;
