const mapJobData = (data?: ApiResponseType) => {
  const apiResponse = data;
  const jobName = apiResponse?.jobDetails?.JOB_NAME || '';
  const jobDescription = apiResponse?.jobDetails?.DESCRIPTION || '';
  const jobType = apiResponse?.jobDetails?.JOB_TYPE || '';

  const operationConfigRows: OperationConfigRow[] = Object.values(apiResponse?.operationConfigurationDetails || {}).map((operation: any) => ({
    operationSequence: operation?.OPERATION_ID,
    operationName: operation?.OPERATION_NAME,
    operationTag: operation?.operation_type, // Adjust based on actual data
    operationTagValue: operation?.OPERATION_TAG_VALUE ?? operation?.OPERATION_TAG_VALUE_GUI,
    operationType: operation?.OPERATION_TYPE
  }));

  jobContext?.updateJobData({
    jobName,
    jobDescription,
    jobType,
    operationConfigRows
  });
  appContext?.updateIsLoading(false);
};

export interface JobContextType {
  jobName: string;
  jobDescription: string;
  jobType: string;
  operationConfigRows?: OperationConfigRow[];
}

export interface OperationConfigRow {
  operationSequence: string;
  operationName: string;
  operationTag: string;
  operationTagValue: string;
  operationType: string;
}