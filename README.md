import { useTranslation } from 'react-i18next';
import React from 'react';
import { Box, Button, Typography } from '@mui/material';

import { DagHeaderProps } from '../../dag.interface';

import themeFile from '../../../../styles/theme.json';
import { useDagContext } from '../DagContext';
import { useValidation } from '../../../../utils/validation';

const DagHeader: React.FC<DagHeaderProps> = ({ onClose, onBack, step }) => {
  const { t } = useTranslation();

  const dagContext = useDagContext();

  const { validateDagDetails, validateDagParams } = dagContext;

  const { triggerValidation } = useValidation(validateDagDetails, validateDagParams);

  const handleNextClick = async () => {
    try {
      await triggerValidation();
      dagContext?.updateDagStage(dagContext?.step + 1);
    } catch (errors) {
      alert(`${t('validation_error')} ${errors}`);
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
