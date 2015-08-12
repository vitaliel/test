# Indexing files from connectors

Flow:

* Breakout fire event for file download after metadata sync
* Dropbox/Box/Gdrive downloader downloads file and file s3 upload event
* S3 uploader uploads file to S3 and fire event to index file
* Indexer puts the record into ES index portfilio_files

## Download file from remote service

Request routing keys:

* dropbox_connector.sync.s3.requested
* box_connector.sync.s3.requested
* gdrive_connector.sync.s3.requested

Message
```
{
  external_uid: '124s4',
  assembla_user_id: 'USER_ID',
  oauth_token: 'secret-oauth-token',
  connector: {
    id: 125,
    portfolio_id: 'UUID',
    space_id: 'UUID',
    space_tool_id: 'UUID',
    path: '/projects/design'
  },
  files: {
    's3key1' => {
      id: 124,
      full_path: 'ticket1/sample2.jpg',
      mime_type: 'image/jpeg',
      size: 123456,
      revision: 'beef'
    }
    's3key2' => {
      id: 142,
      full_path: 'ticket2/sample2.jpg',
      mime_type: 'image/jpeg',
      size: 1234567,
      revision: 'c0ffee'
    }
  }
}
```

Response routing key: *breakout.file_connector_node.download.succeed*

Message
```
{
  external_uid: '124s4',
  assembla_user_id: 'USER_ID',
  oauth_token: 'secret-oauth-token',
  connector: {
    id: 125,
    portfolio_id: 'UUID',
    space_id: 'UUID',
    space_tool_id: 'UUID',
    path: '/projects/design'
  },
  files: {
    's3key1' => {
      id: 124,
      full_path: 'ticket1/sample2.jpg',
      local_path: '/mnt/gluster/dropbox/124.jpg'
      mime_type: 'image/jpeg',
      size: 123456,
      revision: 'beef'
    }
    's3key2' => {
      id: 142,
      full_path: 'ticket2/sample2.jpg',
      local_path: '/mnt/gluster/dropbox/142.jpg'
      mime_type: 'image/jpeg',
      size: 1234567,
      revision: 'c0ffee'
    }
  }
}
```

## S3 uploader response

Routing key: *breakout.file_connector_node.upload.succeed*

Message is the same as for downloader, except it does not contain files that failed to upload for some reason.

## ES indexer

It just listen to message from s3 uploader and put files into ES index
