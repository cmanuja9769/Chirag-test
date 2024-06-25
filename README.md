import { useTranslation } from 'react-i18next';
import React, { useEffect, useState } from 'react';
import { Box, TextField, FormControl, Select, MenuItem, Typography, FormHelperText, Grid } from '@mui/material';
import { Field, Form, Formik } from 'formik';
import { useJobContext } from '../../../../context/jobContext';
import * as Yup from 'yup';
import themeFile from '../../../../styles/theme.json';
import TooltipComponent from '../../../Common/Tooltip';
import { useValidationContext } from '../../../../context/validationContext';

const JobDetails: React.FC = () => {
  const { t } = useTranslation();
  const { jobData, updateJobData, setJobDetailsValidation } = useJobContext();
  const { registerValidator } = useValidationContext();

  const [charCount, setCharCount] = useState(jobData?.description?.length || 0);
  const [formTouched, setFormTouched] = useState(false); // Track if form has been touched

  const validationSchema = Yup.object({
    job_name: Yup.string().min(2, 'Too Short!').max(50, 'Too Long!').required(t('dag_name_reqd')),
    description: Yup.string().min(2, 'Too Short!').max(256, 'Too Long!').required(t('dag_description_reqd')),
    job_type: Yup.string().required(t('create_dag_status_reqd'))
  });

  const handleChange = (e?: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e?.target;
    updateJobData((prevJobData?) => ({
      ...prevJobData,
      [name]: value
    }));
  };

  useEffect(() => {
    setJobDetailsValidation(() => async () => {
      try {
        await validationSchema.validate(jobData, { abortEarly: false });
        return Promise.resolve();
      } catch (errors) {
        return Promise.reject(errors);
      }
    });
  }, [jobData, setJobDetailsValidation]);

  useEffect(() => {
    registerValidator(() => validationSchema.validate(jobData));
  }, [jobData, registerValidator]);

  const handleSubmit = async (values, { setSubmitting, setFieldTouched }) => {
    // Manually mark all fields as touched
    Object.keys(values).forEach(field => setFieldTouched(field));

    try {
      await updateJobData(values);
      setSubmitting(false);
    } catch (error) {
      console.error('Error updating job data:', error);
      setSubmitting(false);
    }
  };

  return (
    <Box
      sx={{
        border: `2px solid ${themeFile?.colors?.pfizerBlue}`,
        mt: 5,
        display: 'block',
        paddingBottom: 2
      }}
    >
      <Typography
        variant="h6"
        component="h6"
        padding={1}
        style={{ background: themeFile?.colors?.pfizerBlue, color: themeFile?.colors?.white, fontWeight: 'bold' }}
      >
        {t('job_det_label')}
      </Typography>
      <Formik
        initialValues={jobData}
        validationSchema={validationSchema}
        validateOnChange={false}
        validateOnBlur={true}
        onSubmit={handleSubmit}
      >
        {({ values, handleBlur, errors, touched, setFieldValue, setFieldTouched, isSubmitting }) => (
          <Form>
            <Box display="inline-flex" alignItems="center" padding={2}>
              <Grid container spacing={2} direction="row" alignItems="center" justifyContent="center">
                <Grid item xs={12} sm={12} display="inline-flex" justifyContent="space-around" alignItems="center">
                  <Grid item xs={12} sm={2} marginTop="1vh" display="inline-flex">
                    <Typography sx={{ fontSize: 15, fontWeight: 'bold' }}>{t('job_det_name')}</Typography>
                    <TooltipComponent title={t('job_det_name')} />
                  </Grid>
                  <Grid item xs={12} sm={5}>
                    <Field
                      sx={{ width: '20vw' }}
                      as={TextField}
                      placeholder="Job Name"
                      name="job_name"
                      size="small"
                      value={values?.job_name}
                      onChange={(e: React.ChangeEvent<HTMLInputElement>) => {
                        setFieldValue('job_name', e?.target?.value);
                        handleChange(e);
                      }}
                      onBlur={handleBlur}
                      margin="normal"
                      error={(touched?.job_name || formTouched) && Boolean(errors?.job_name)}
                    />
                    {(touched?.job_name || formTouched) && errors?.job_name && (
                      <FormHelperText error>{errors?.job_name}</FormHelperText>
                    )}
                  </Grid>
                  <Grid item xs={12} sm={1} marginTop="1vh" display="inline-flex">
                    <Typography sx={{ fontSize: 15, fontWeight: 'bold' }}>{t('create_job_type')}</Typography>
                    <TooltipComponent title={t('create_job_type')} />
                  </Grid>
                  <Grid item xs={12} sm={4}>
                    <FormControl margin="normal" error={(touched.job_type || formTouched) && Boolean(errors.job_type)}>
                      <Field
                        sx={{ width: '20vw' }}
                        as={Select}
                        name="job_type"
                        value={values?.job_type}
                        onChange={(e: React.ChangeEvent<{ value: unknown }>) => {
                          setFieldValue('job_type', e?.target?.value as string);
                          handleChange(e as React.ChangeEvent<HTMLInputElement>);
                        }}
                        onBlur={(e: React.FocusEvent) => {
                          handleBlur(e);
                          setFieldTouched('job_type', true);
                        }}
                      >
                        <MenuItem value={t('job_det_status_snowflake')}>{t('job_det_status_snowflake')}</MenuItem>
                        <MenuItem value={t('job_det_status_databrick')}>{t('job_det_status_databrick')}</MenuItem>
                        <MenuItem value={t('job_det_status_trans')}>{t('job_det_status_trans')}</MenuItem>
                      </Field>
                      {(touched?.job_type || formTouched) && errors?.job_type && (
                        <FormHelperText error>{errors?.job_type}</FormHelperText>
                      )}
                    </FormControl>
                  </Grid>
                </Grid>

                <Grid item xs={12} sm={2} margin="auto" display="inline-flex">
                  <Typography sx={{ fontSize: 15, fontWeight: 'bold' }}>{t('job_description')}</Typography>
                  <TooltipComponent title={t('job_description')} />
                </Grid>
                <Grid item xs={12} sm={10}>
                  <Field
                    sx={{ width: '25vw' }}
                    as={TextField}
                    placeholder="Description"
                    name="description"
                    multiline
                    rows={4}
                    value={values?.description}
                    onChange={(e: React.ChangeEvent<HTMLInputElement>) => {
                      if (e?.target?.value?.length <= 256) {
                        setFieldValue('description', e?.target?.value);
                        setCharCount(e?.target?.value?.length);
                        handleChange(e);
                      }
                    }}
                    onBlur={handleBlur}
                    margin="normal"
                    error={(touched?.description || formTouched) && Boolean(errors?.description)}
                  />
                  {(touched?.description || formTouched) && errors?.description && (
                    <FormHelperText error>{errors?.description}</FormHelperText>
                  )}
                  {!errors?.description && (
                    <FormHelperText>{`${256 - charCount} characters remaining`}</FormHelperText>
                  )}
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
