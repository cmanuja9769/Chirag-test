<Box key={header.name} width="100%">
                          <div className="tw-mb-2 tw-relative">
                            <div className="tw-flex tw-items-center">
                              <div className="tw-flex tw-justify-between tw-items-center tw-mr-4 tw-w-[145px]">
                                <Typography variant="body1" className="tw-font-medium">
                                  {header?.label}:{header?.validation && <span style={{ color: 'red' }}> *</span>}
                                </Typography>
                                {header?.tooltipText && (
                                  <Tooltip title={header?.tooltipText}>
                                    <IconButton size="small">
                                      <img id="target" alt="tooltip" src={Info} />
                                    </IconButton>
                                  </Tooltip>
                                )}
                              </div>
                              <div className="tw-flex-1">
                                <TextField
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
                                  variant="outlined"
                                  error={!!errors[header?.name] && touched[header?.name]}
                                  type={header?.type}
                                />
                              </div>
                            </div>
                            {touched[header.name] && errors[header.name] && (
                              <Typography color="error" variant="caption" className="tw-absolute tw-top-full tw-left-[10rem]">
                                {errors[header.name] || 'Hello'}
                              </Typography>
                            )}
                            {header?.name === 'description' && (
                              <Typography variant="caption" className="tw-absolute tw-top-full tw-right-[27rem]">
                                {`${MAX_DESCRIPTION_LENGTH - values[header.name].length} characters remaining`}
                              </Typography>
                            )}
                          </div>
                        </Box>
