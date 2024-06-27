import { useTranslation } from 'react-i18next';
import React, { useState } from 'react';
import {
  Accordion,
  AccordionSummary,
  AccordionDetails,
  Typography,
  Select,
  MenuItem,
  FormControl,
  Checkbox,
  FormControlLabel,
  Button,
  Modal,
  TextField,
  SelectChangeEvent,
  Box,
  Grid
} from '@mui/material';
import ExpandMoreIcon from '@mui/icons-material/ExpandMore';
import { useJobContext } from '../../../../context/jobContext';
import { TOAST_TYPE } from '../../../../utils/enums';
import ToastMessage from '../../../Common/ToastMessage/ToastMessage';

const DatasetConfiguration: React.FC = () => {
  const { t } = useTranslation();
  const { jobData, updateJobData } = useJobContext();
  const [openModal, setOpenModal] = useState(false);
  const [previewModalOpen, setPreviewModalOpen] = useState(false);
  const [currentCheckbox, setCurrentCheckbox] = useState('');
  const [openToast, setOpenToast] = useState<boolean>(false);
  const [toastInfo, setToastInfo] = useState<{ severity: TOAST_TYPE; message: string }>({ message: '', severity: TOAST_TYPE.INFO });

  const options = [
    { value: t('merge_val'), checkboxes: [t('join_val'), t('delta_val')] },
    { value: t('t&l_val'), checkboxes: [t('join_val'), t('filter_val'), t('group_val'), t('distinct_val')] },
    { value: t('append_val'), checkboxes: [t('join_val'), t('filter_val'), t('group_val')] },
    { value: t('upd_val'), checkboxes: [t('join_val'), t('delta_val')] }
  ];

  const handleToast = (message: string, type: TOAST_TYPE) => {
    setOpenToast(true);
    setToastInfo({
      message: message,
      severity: type
    });
  };

  const handleSelectChange = (event: SelectChangeEvent) => {
    const selectedOption = event.target.value as string;
    updateJobData((prevState) => ({
      ...prevState,
      selectedOption,
      checkboxStates: {}
    }));
  };

  const handleCheckboxChange = (checkbox: string) => {
    if (checkbox === t('distinct_val')) {
      updateJobData((prevState) => ({
        ...prevState,
        checkboxStates: {
          ...prevState.checkboxStates,
          [checkbox]: !prevState.checkboxStates[checkbox]
        }
      }));
    } else {
      setCurrentCheckbox(checkbox);
      setOpenModal(true);
    }
  };

  const handleModalSave = () => {
    if (jobData.previewData[currentCheckbox]) {
      updateJobData((prevState) => ({
        ...prevState,
        checkboxStates: {
          ...prevState.checkboxStates,
          [currentCheckbox]: true
        }
      }));
      setOpenModal(false);
    } else {
      handleToast(t('form_error'), TOAST_TYPE.ERROR);
    }
  };

  const handleModalCancel = () => {
    setOpenModal(false);
  };

  const handlePreviewButtonClick = () => {
    setPreviewModalOpen(true);
  };

  const renderCheckboxes = () => {
    const selectedOption = options.find((option) => option.value === jobData.selectedOption);
    return selectedOption?.checkboxes.map((checkbox) => (
      <Grid container spacing={3} direction="row" alignItems="center" justifyContent="center" key={checkbox}>
        <Grid item xs={12} sm={12} display="inline-flex" alignItems="center">
          <Grid item xs={12} sm={4} marginTop="3vh" display="inline-flex">
            <FormControlLabel
              control={<Checkbox checked={jobData.checkboxStates[checkbox] || false} onChange={() => handleCheckboxChange(checkbox)} />}
              label={checkbox}
              disabled={checkbox === t('distinct_val')}
            />
          </Grid>
          {checkbox !== t('distinct_val') && (
            <Grid item xs={12} sm={8} marginTop="3vh">
              <Button variant="contained" disabled={!jobData.checkboxStates[checkbox]} onClick={handlePreviewButtonClick}>
                {t('preview_config_lbl')}
              </Button>
            </Grid>
          )}
        </Grid>
      </Grid>
    ));
  };

  return (
    <Box className="tw-mt-5 tw-block tw-pb-2">
      <Accordion defaultExpanded={false} className="tw-border-solid tw-border-2 tw-border-pfizerBlue">
        <AccordionSummary
          expandIcon={<ExpandMoreIcon className="tw-text-white tw-pe-2 tw-text-[2vw]" />}
          className="tw-bg-pfizerBlue tw-text-white tw-font-bold tw-ps-1"
        >
          <Typography variant="h6" component="h6" paddingLeft={1}>
            {t('dataset_lvl_lbl')}
          </Typography>
        </AccordionSummary>
        <Box display="inline-flex" alignItems="center" padding={2}>
          <AccordionDetails className="tw-pt-2">
            <Grid container spacing={1} direction="row" alignItems="center" justifyContent="center">
              <Grid item xs={12} sm={12} display="inline-flex" alignItems="center">
                <Grid item xs={12} sm={2} marginTop="2vh" display="inline-flex">
                  <Typography sx={{ fontSize: 15, fontWeight: 'bold' }}>{t('operation_lbl')}</Typography>
                </Grid>
                <Grid item xs={12} sm={4} className="tw-ms-10">
                  <FormControl margin="normal">
                    <Select sx={{ width: '20vw' }} value={jobData?.selectedOption} onChange={handleSelectChange}>
                      {options?.map((option?) => (
                        <MenuItem key={option?.value} value={option?.value}>
                          {option?.value}
                        </MenuItem>
                      ))}
                    </Select>
                  </FormControl>
                </Grid>
              </Grid>
            </Grid>
          </AccordionDetails>
        </Box>
        <Box alignItems="center" className="tw-ps-10 tw-mt-5">
          {renderCheckboxes()}
        </Box>
      </Accordion>
      <Modal open={openModal} onClose={() => setOpenModal(false)}>
        <Box sx={{ padding: '20px', backgroundColor: 'white', margin: '20px auto', width: '300px' }}>
          <Typography variant="h6">
            {currentCheckbox} {t('configuration_lbl')}
          </Typography>
          <TextField
            fullWidth
            value={jobData.previewData[currentCheckbox] || ''}
            onChange={(e) =>
              updateJobData((prevState) => ({
                ...prevState,
                previewData: {
                  ...prevState.previewData,
                  [currentCheckbox]: e.target.value
                }
              }))
            }
          />
          <Button onClick={handleModalSave}>{t('save')}</Button>
          <Button onClick={handleModalCancel}>{t('cancel')}</Button>
        </Box>
      </Modal>
      <Modal open={previewModalOpen} onClose={() => setPreviewModalOpen(false)}>
        <Box sx={{ padding: '20px', backgroundColor: 'white', margin: '20px auto', width: '300px' }}>
          <Typography variant="h6">{t('preview_config_lbl')}</Typography>
          <Typography>{jobData.previewData[currentCheckbox] || t('no_data_lbl')}</Typography>
          <Button onClick={() => setPreviewModalOpen(false)}>{t('close')}</Button>
        </Box>
      </Modal>
      <ToastMessage severity={toastInfo.severity} isVisible={openToast} hideToast={() => setOpenToast(false)} message={toastInfo.message} />;
    </Box>
  );
};

export default DatasetConfiguration;
