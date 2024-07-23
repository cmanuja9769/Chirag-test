import { useEffect, useState } from 'react';

interface ApiResponseType {
  jobDetails?: {
    JOB_ID?: string;
    JOB_NAME?: string;
    DESCRIPTION?: string;
    JOB_TYPE?: string;
    JOB_CATEGORY?: string;
    ACTIVE_FLAG?: string;
  };
  operationConfigurationDetails?: Array<{
    OPERATION_ID?: string;
    OPERATION_NAME?: string;
    OPERATION_TYPE?: string;
    OPERATION_TAG?: string;
    OPERATION_TAG_VALUE?: string;
    OPERATION_TAG_VALUE_GUI?: string;
  }>;
}

const MyComponent = () => {
  const [rows, setRows] = useState<{ id: number; [key: string]: string }[]>([]);
  const jobContext = {
    jobData: {
      operationConfigRows: [], // Replace with actual job data
    },
    updateJobData: (data: any) => {
      // Update job data function implementation
    }
  };
  const appContext = {
    updateIsLoading: (isLoading: boolean) => {
      // Update loading state function implementation
    }
  };

  useEffect(() => {
    const formattedRows = jobContext?.jobData?.operationConfigRows?.map((row, index) => ({
      id: index,
      ...row
    })) || [];
    setRows(formattedRows);
  }, [jobContext?.jobData?.operationConfigRows]);

  const mapJobData = (data?: ApiResponseType) => {
    if (!data) return;

    const jobName = data?.jobDetails?.JOB_NAME || '';
    const jobDescription = data?.jobDetails?.DESCRIPTION || '';
    const jobType = data?.jobDetails?.JOB_TYPE || '';

    const operationConfigRows = data.operationConfigurationDetails?.map((item) => ({
      operationSequence: item?.OPERATION_ID || '',
      operationName: item?.OPERATION_NAME || '',
      operationTag: item?.OPERATION_TAG || '',
      operationTagValue: item?.OPERATION_TAG_VALUE_GUI ?? item?.OPERATION_TAG_VALUE || '',
      operationType: item?.OPERATION_TYPE || ''
    })) || [];

    console.log('rows', operationConfigRows);

    jobContext?.updateJobData({
      jobName,
      jobDescription,
      jobType,
      operationConfigRows
    });
    appContext?.updateIsLoading(false);
  };

  return (
    // Your component JSX
  );
};