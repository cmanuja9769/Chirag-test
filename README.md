<FormControl fullWidth margin="normal" error={touched.jobType && Boolean(errors?.jobType)}>
  <Field
    as={Select}
    sx={{ padding: '0.3rem' }}
    size="medium"
    name="jobType"
    value={values?.jobType || ''}
    onChange={(e: ChangeEvent<{ value?: any }>) => {
      setFieldValue('jobType', e?.target?.value as string);
      handleJobTypeChange(e as ChangeEvent<HTMLInputElement>);
    }}
    onBlur={handleBlur}
    endAdornment={
      <InputAdornment sx={{ marginRight: '2.5rem' }} position="start">
        {fetchingJobTypes && <CircularProgress size={20} />}
      </InputAdornment>
    }
  >
    {jobCtx?.jobTypes?.length ? (
      jobCtx?.jobTypes?.map((jobType) => (
        <MenuItem
          key={typeof jobType?.ENTITY_KEYS === 'string' ? jobType?.ENTITY_KEYS : jobType?.ENTITY_KEYS?.id}
          value={typeof jobType?.ENTITY_KEYS === 'string' ? jobType.ENTITY_KEYS : jobType?.ENTITY_KEYS?.id}
        >
          {jobType?.ENTITY_NAME}
        </MenuItem>
      ))
    ) : (
      <MenuItem value={values?.jobType}>{values?.jobType}</MenuItem>
    )}
  </Field>
  {touched?.jobType && errors?.jobType && (
    <FormHelperText className="tw-absolute tw-bottom-[-1rem]">{errors?.jobType}</FormHelperText>
  )}
</FormControl>