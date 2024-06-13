// useValidation.ts
import { useCallback } from 'react';

export const useValidation = (validateDagDetails: () => Promise<void>, validateDagParams: () => Promise<void>) => {
  const triggerValidation = useCallback(async () => {
    try {
      await Promise.all([validateDagDetails(), validateDagParams()]);
      return Promise.resolve();
    } catch (errors) {
      return Promise.reject(errors);
    }
  }, [validateDagDetails, validateDagParams]);

  return { triggerValidation };
};

// DagHeader.tsx
import React from 'react';
import { Box, Button, Typography } from '@mui/material';
import { useTranslation } from 'react-i18next';
import { useFormikContext } from 'formik';
import { DagHeaderProps } from '../../dag.interface';
import { useDagContext } from '../DagContext';
import { useValidation } from '../../../../utils/validation';
import themeFile from '../../../../styles/theme.json';

const DagHeader: React.FC<DagHeaderProps> = ({ onClose, onBack, step }) => {
  const { t } = useTranslation();
  const { validateDagDetails, validateDagParams } = useDagContext();
  const { triggerValidation } = useValidation(validateDagDetails, validateDagParams);
  const formikContext = useFormikContext();

  const handleNextClick = async () => {
    try {
      await triggerValidation();
      formikContext.setTouched({});
      formikContext.validateForm().then((errors) => {
        if (Object.keys(errors).length > 0) {
          throw new Error('Form validation failed');
        }
        useDagContext()?.updateDagStage(step + 1);
      });
    } catch (error) {
      alert(`${t('validation_error')} ${error}`);
    }
  };

  return (
    <Box component="header" display="flex" justifyContent="space-between" padding={1}>
      <Typography variant="h5" component="h5" padding={1} style={{ color: themeFile?.colors?.pfizerBlue, fontWeight: 'bold' }}>
        {t('create_new_dag')}
      </Typography>
      <Box>
        {step > 0 && (
          <Button variant="outlined" color="primary" onClick={onBack} style={{ marginRight: '8px' }}>
            {t('back')}
          </Button>
        )}
        <Button variant="contained" color="primary" onClick={handleNextClick}>
          {t('next')}
        </Button>
        <Button variant="outlined" color="secondary" onClick={onClose} style={{ marginLeft: '8px' }}>
          {t('close')}
        </Button>
      </Box>
    </Box>
  );
};

export default DagHeader;