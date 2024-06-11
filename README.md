  <Formik
        initialValues={dagData}
        validationSchema={validationSchema}
        onSubmit={(values) => {
          updateDagData(values);
          console.log('Form Submitted with values:', values);
        }}
        enableReinitialize
        validateOnChange={false}
        validateOnBlur={true}
      >
        {({ values, handleBlur, setFieldValue, errors, touched }) => (
          <Form id="dag-details-form">
            <Box display="inline-flex" alignItems="center" padding={2} sx={{ '& .MuiFormControl-root': { width: '30vw' } }}>
              <Grid container spacing={3} direction="row" alignItems="center" justifyContent="center">
                {headersData
                  .sort((a, b) => a.sequence - b.sequence)
                  .map((header) => (
                    <Grid item xs={12} sm={12} key={header.name}>
                      {header.name === 'description' ? (
                        <Box key={header.name} width="100%">
                          <TextField
                            label={header.label}
                            value={values[header.name]}
                            onChange={(e) => {
                              const value = e.target.value;
                              if (value.length <= MAX_DESCRIPTION_LENGTH) {
                                setFieldValue(header.name, value);
                                updateDagData((prevData) => ({
                                  ...prevData,
                                  [header.name]: value
                                }));
                              }
                            }}
                            onBlur={handleBlur}
                            fullWidth
                            multiline
                            rows={4}
                            error={Boolean(touched[header.name] && errors[header.name])}
                            helperText={
                              touched[header.name] && errors[header.name]
                                ? errors[header.name]
                                : `${MAX_DESCRIPTION_LENGTH - values[header.name].length} characters remaining`
                            }
                          />
                        </Box>
                      ) : (
                        <DynamicComponent
                          key={header.name}
                          fieldData={header}
                          value={values[header.name]}
                          onChange={(value) => {
                            setFieldValue(header.name, value);
                            updateDagData((prevData) => ({
                              ...prevData,
                              [header.name]: value
                            }));
                          }}
                          error={errors[header.name]}
                          touched={touched[header.name]}
                          disabled={false}
                        />
                      )}
                    </Grid>
                  ))}
              </Grid>
            </Box>
          </Form>
        )}
      </Formik>