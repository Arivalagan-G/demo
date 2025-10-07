            Delete Selected
          </Button>
          <Button
            type="primary"
            onClick={() => (window.location.href = '/watchlist')}
          >
            View Watchlist ❤️
          </Button>
        </Space>
      </div>

      {/* Image Grid */}
      <Row gutter={[16, 16]}>
        {filteredPhotos.map((photo) => (
          <Col key={photo.id} xs={24} sm={12} md={8} lg={6}>
            <Card
              hoverable
              cover={
                <div style={{ position: 'relative' }}>
                  <LazyLoadImage
                    src={photo.src}
                    alt={`Image #${photo.id}`}
                    effect="blur"
                    style={{
                      width: '100%',
                      height: 200,
                      objectFit: 'cover',
                      borderRadius: 8,
                    }}
                    onClick={() => openModal(photo)}
                  />
                  <Checkbox
                    checked={selected.has(photo.id)}
                    onChange={() => toggleSelect(photo.id)}
                    style={{
                      position: 'absolute',
                      bottom: 8,
                      left: 8,
                      transform: 'scale(1.3)',
                    }}
                  />
                </div>
              }
            >
              {/* Show image number instead of author */}
              <Card.Meta title={`Image #${photo.id}`} />

              {/* Watchlist heart below title */}
              <div style={{ textAlign: 'center', marginTop: 8 }}>
                <Tooltip title="Add to Watchlist">
                  <Button
                    shape="circle"
                    size="small"
                    icon={
                      watchlist.some((p) => p.id === photo.id) ? (
                        <HeartFilled style={{ color: 'red' }} />
                      ) : (
                        <HeartOutlined style={{ color: 'gray' }} />
                      )
                    }
                    onClick={() => toggleWatchlist(photo)}
                  />
                </Tooltip>
              </div>
            </Card>
          </Col>
        ))}
      </Row>

      {/* Scroll to top */}
      <FloatButton
        icon={<ArrowUpOutlined />}
        tooltip="Go to top"
        onClick={() => window.scrollTo({ top: 0, behavior: 'smooth' })}
      />

      {/* Image Modal */}
      <Modal
        open={!!modalPhoto}
        onCancel={() => setModalPhoto(null)}
        footer={null}
        width={800}
      >
        {modalPhoto && (
          <div style={{ textAlign: 'center' }}>
            <img
              src={modalPhoto.src}
              alt={`Image #${modalPhoto.id}`}
              style={{
                maxWidth: '100%',
                transform: `rotate(${rotation}deg)`,
                transition: 'transform 0.3s',
              }}
            />
            <Space style={{ marginTop: 10 }}>
              <Button icon={<RotateRightOutlined />} onClick={rotateImage}>
                Rotate
              </Button>
              <Button
                icon={<DownloadOutlined />}
                onClick={() => handleDownload(modalPhoto.src)}
              >
                Download
              </Button>
              <Button
                icon={<ShareAltOutlined />}
                onClick={() => handleShare(modalPhoto.src)}
              >
                Share
              </Button>
              <Button
                icon={<PrinterOutlined />}
                onClick={() => handlePrint(modalPhoto.src)}
              >
                Print
              </Button>
              <Button
                danger
                icon={<DeleteOutlined />}
                onClick={() => {
                  setPhotos((prev) =>
                    prev.filter((p) => p.id !== modalPhoto.id)
                  );
                  setModalPhoto(null);
                  message.success('Image deleted');
                }}
              >
                Delete
              </Button>
            </Space>
          </div>
        )}
      </Modal>
    </div>
  );
}'use client';
import React, { useState, useEffect } from 'react';
import { Card, Row, Col, Button, message } from 'antd';
import { ArrowLeftOutlined } from '@ant-design/icons';

export default function WatchlistPage() {
  const [watchlist, setWatchlist] = useState([]);

  useEffect(() => {
    const stored = JSON.parse(localStorage.getItem('watchlist')) || [];
    setWatchlist(stored);
  }, []);

  const handleRemove = (id) => {
    const updated = watchlist.filter((img) => img.id !== id);
    setWatchlist(updated);
    localStorage.setItem('watchlist', JSON.stringify(updated));
    message.success('Removed from watchlist');
  };

  return (
    <div style={{ padding: 20 }}>
      <Button
        icon={<ArrowLeftOutlined />}
        onClick={() => (window.location.href = '/')}
        style={{ marginBottom: 10 }}
      >
        Back to Album
      </Button>
      <h1>❤️ My Watchlist</h1>
      <Row gutter={[16, 16]}>
        {watchlist.map((img) => (
          <Col key={img.id} xs={24} sm={12} md={8} lg={6}>
            <Card
              hoverable
              cover={
                <img
                  src={img.src}
                  alt={img.author}
                  style={{
                    width: '100%',
                    height: 200,
                    objectFit: 'cover',
                    borderRadius: 8,
                  }}
                />
              }
              actions={[
                <Button danger onClick={() => handleRemove(img.id)}>
                  Remove
                </Button>,
              ]}
            >
              <Card.Meta title={img.author} />
            </Card>
          </Col>
        ))}
      </Row>
    </div>
  );
}
