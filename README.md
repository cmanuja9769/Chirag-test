import { useState, useCallback } from 'react';

const useValidation = (validateDagDetails: () => Promise<void>, validateDagParams: () => Promise<void>) => {
  const triggerValidation = useCallback(async () => {
    try {
      await validateDagDetails();
      await validateDagParams();
      return Promise.resolve();
    } catch (errors) {
      return Promise.reject(errors);
    }
  }, [validateDagDetails, validateDagParams]);

  return { triggerValidation };
};

export default useValidation;