const formikContext = useFormikContext<FormikValues>(); // Get Formik context

  const handleNextClick = async () => {
    try {
      await triggerValidation(formikContext, formikContext.initialValues); // Pass the formik context and initialValues
const useValidation = (validateDagDetails: () => Promise<void>, validateDagParams: () => Promise<void>) => {
  const triggerValidation = useCallback(
    async (formikHelpers: FormikHelpers<FormikValues>, initialValues: FormikValues) => {
      try {
        formikHelpers.setTouched(
          Object.keys(initialValues).reduce((acc, field) => {
            acc[field] = true;
            return acc;
          }, {} as Record<string, boolean>)
        );
        await validateDagDetails();
        await validateDagParams();
        return Promise.resolve();
      } catch (errors) {
        return Promise.reject(errors);
      }
    },
    [validateDagDetails, validateDagParams]
  );

  return { triggerValidation };
};
