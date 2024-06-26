import { useTranslation } from 'react-i18next';
import React, { useEffect, useState } from 'react';
import { Box, TextField, FormControl, Select, MenuItem, Typography, FormHelperText, Grid } from '@mui/material';

import { Field, Form, Formik, FormikHelpers, FormikProps } from 'formik';
import { useJobContext } from '../../../../context/jobContext';

import * as Yup from 'yup';

import themeFile from '../../../../styles/theme.json';

import TooltipComponent from '../../../Common/Tooltip';
import { JobData } from '../../job.interface';

interface MyFormProps {
  formRef: React.RefObject<FormikProps<JobData>>;
}

const JobDetails: React.FC<MyFormProps> = ({ formRef }) => {
  const { t } = useTranslation();
  const { jobData, updateJobData, setJobDetailsValidation, isNexButtonClicked } = useJobContext();
  const [charCount, setCharCount] = useState(jobData?.description?.length || 0);

  const validationSchema = Yup.object({
    job_name: Yup.string().min(2, 'Too Short!').max(50, 'Too Long!').required(t('dag_name_reqd')),
    description: Yup.string().min(2, 'Too Short!').max(50, 'Too Long!').required(t('dag_description_reqd')).max(256, t('no_chars')),
    job_type: Yup.string().required(t('create_dag_status_reqd'))
  });

  const handleChange = (e?: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e?.target;
    updateJobData((prevJobData?) => ({
      ...prevJobData,
      [name]: value
    }));
  };

  const handleSubmit = (values: JobData, actions: FormikHelpers<JobData>) => {
    console.log('Values', values);
    actions.setSubmitting(false);
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
    if (isNexButtonClicked) {
      if (formRef?.current) {
        formRef?.current?.submitForm();
        const errors = formRef?.current?.validateForm();
        if (Object.keys(errors).length === 0) {
          console.log('No errors from job details', Object.keys(errors).length);
        }
      }
    }
  }, [isNexButtonClicked]);

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
        innerRef={formRef}
        initialValues={jobData}
        validationSchema={validationSchema}
        validateOnChange={false}
        validateOnBlur={true}
        onSubmit={(values, { setSubmitting }) => {
          updateJobData(values);
          setSubmitting(false);
          console.log('values', values);
        }}
      >
        {({ values, handleBlur, errors, touched, setFieldValue }) => (
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
                      error={touched?.job_name && Boolean(errors?.job_name)}
                      helperText={touched?.job_name && errors?.job_name}
                      FormHelperTextProps={{ style: { position: 'absolute', bottom: '-20px' } }}
                    />
                  </Grid>
                  <Grid item xs={12} sm={1} marginTop="1vh" display="inline-flex">
                    <Typography sx={{ fontSize: 15, fontWeight: 'bold' }}>{t('create_job_type')}</Typography>
                    <TooltipComponent title={t('create_job_type')} />
                  </Grid>
                  <Grid item xs={12} sm={4}>
                    <FormControl margin="normal" error={touched.job_type && Boolean(errors.job_type)}>
                      <Field
                        sx={{ width: '20vw' }}
                        as={Select}
                        name="job_type"
                        value={values?.job_type}
                        onChange={(e: React.ChangeEvent<{ value: unknown }>) => {
                          setFieldValue('job_type', e?.target?.value as string);
                          handleChange(e as React.ChangeEvent<HTMLInputElement>);
                        }}
                        onBlur={handleBlur}
                      >
                        <MenuItem value={t('job_det_status_snowflake')}>{t('job_det_status_snowflake')}</MenuItem>
                        <MenuItem value={t('job_det_status_databrick')}>{t('job_det_status_databrick')}</MenuItem>
                        <MenuItem value={t('job_det_status_trans')}>{t('job_det_status_trans')}</MenuItem>
                      </Field>
                      {touched?.job_type && errors?.job_type && (
                        <FormHelperText style={{ position: 'absolute', bottom: '-20px' }}>{errors.job_type}</FormHelperText>
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
                    error={touched?.description && Boolean(errors?.description)}
                    helperText={(touched?.description && errors?.description) || `${256 - charCount} characters remaining`}
                    FormHelperTextProps={{ style: { position: 'absolute', bottom: '-20px' } }}
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
