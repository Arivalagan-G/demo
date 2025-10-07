'use client';
import React, { useState, useEffect } from 'react';
import {
  Row,
  Col,
  Card,
  Image,
  Button,
  Tooltip,
  Space,
  Modal,
  FloatButton,
  Checkbox,
  Select,
  message,
} from 'antd';
import {
  DownloadOutlined,
  ShareAltOutlined,
  DeleteOutlined,
  RotateRightOutlined,
  PrinterOutlined,
  HeartOutlined,
  HeartFilled,
  ArrowUpOutlined,
} from '@ant-design/icons';
import { LazyLoadImage } from 'react-lazy-load-image-component';
import JSZip from 'jszip';
import { saveAs } from 'file-saver';
import 'react-lazy-load-image-component/src/effects/blur.css';
import 'antd/dist/reset.css';

export default function PhotoAlbum() {
  const [photos, setPhotos] = useState([]);
  const [selected, setSelected] = useState(new Set());
  const [watchlist, setWatchlist] = useState([]);
  const [modalPhoto, setModalPhoto] = useState(null);
  const [rotation, setRotation] = useState(0);
  const [loading, setLoading] = useState(false);
  const [authorFilter, setAuthorFilter] = useState('All');

  // Add inside PhotoAlbum component, after `const [rotation, setRotation] = useState(0);`

useEffect(() => {
  const handleKeyDown = (e) => {
    if (!modalPhoto) return; // only when modal is open
    if (e.key === 'd' || e.key === 'D') {
      handleDownload(modalPhoto.src, `image-${modalPhoto.id}.jpg`);
    }
    if (e.key === 's' || e.key === 'S') {
      handleShare(modalPhoto.src);
    }
  };

  window.addEventListener('keydown', handleKeyDown);
  return () => window.removeEventListener('keydown', handleKeyDown);
}, [modalPhoto]);

  // Load watchlist from localStorage
  useEffect(() => {
    const savedWatchlist = JSON.parse(localStorage.getItem('watchlist')) || [];
    setWatchlist(savedWatchlist);
  }, []);

  // Save watchlist to localStorage whenever changed
  useEffect(() => {
    localStorage.setItem('watchlist', JSON.stringify(watchlist));
  }, [watchlist]);

  // Generate 500 images
  useEffect(() => {
    const total = 500;
    const newPhotos = Array.from({ length: total }, (_, i) => ({
      id: i,
      author: `Author ${Math.floor(Math.random() * 20) + 1}`,
      src: `https://picsum.photos/id/${i % 1000}/600/400`,
    }));
    setPhotos(newPhotos);
  }, []);

  const authors = ['All', ...Array.from(new Set(photos.map((p) => p.author)))];

  const toggleSelect = (id) => {
    const newSelected = new Set(selected);
    newSelected.has(id) ? newSelected.delete(id) : newSelected.add(id);
    setSelected(newSelected);
  };

  const toggleWatchlist = (photo) => {
    const exists = watchlist.some((p) => p.id === photo.id);
    const updated = exists
      ? watchlist.filter((p) => p.id !== photo.id)
      : [...watchlist, photo];
    setWatchlist(updated);
    message.success(exists ? 'Removed from Watchlist' : 'Added to Watchlist ❤️');
  };

  const handleDownload = async (imageUrl, filename = 'image.jpg') => {
    try {
      const response = await fetch(imageUrl);
      const blob = await response.blob();
      saveAs(blob, filename);
      message.success('Image downloaded!');
    } catch {
      message.error('Download failed.');
    }
  };

  const handleShare = async (imageUrl) => {
    if (navigator.share) {
      try {
        const response = await fetch(imageUrl);
        const blob = await response.blob();
        const file = new File([blob], 'shared_image.jpg', { type: blob.type });
        await navigator.share({ files: [file], title: 'Shared Image' });
      } catch {
        message.error('Share failed.');
      }
    } else {
      navigator.clipboard.writeText(imageUrl);
      message.info('Link copied to clipboard!');
    }
  };

  const handlePrint = (imageUrl) => {
    const printWindow = window.open('', '_blank');
    printWindow.document.write(`
      <html><head><title>Print Image</title></head>
      <body><img src="${imageUrl}" style="max-width:100%" onload="window.print();window.close();" /></body>
      </html>
    `);
    printWindow.document.close();
  };

  const handleBulkDownload = async () => {
    if (!selected.size) return message.warning('Select images first');
    const zip = new JSZip();
    setLoading(true);
    for (const id of selected) {
      const photo = photos.find((p) => p.id === id);
      const response = await fetch(photo.src);
      const blob = await response.blob();
      zip.file(`photo-${photo.id}.jpg`, blob);
    }
    const content = await zip.generateAsync({ type: 'blob' });
    saveAs(content, 'photos.zip');
    setLoading(false);
    message.success('Downloaded selected as ZIP!');
  };

  const handleDeleteSelected = () => {
    if (!selected.size) return message.warning('Select images to delete');
    setPhotos((prev) => prev.filter((p) => !selected.has(p.id)));
    setSelected(new Set());
    message.success('Selected images deleted!');
  };

  const openModal = (photo) => {
    setRotation(0);
    setModalPhoto(photo);
  };

  const rotateImage = () => setRotation((r) => (r + 90) % 360);

  // Filtered Photos
  const filteredPhotos =
    authorFilter === 'All'
      ? photos
      : photos.filter((p) => p.author === authorFilter);

  return (
    <div style={{ padding: 20 }}>
      <h1 style={{ textAlign: 'center' }}>📸 Dynamic Photo Album</h1>

      {/* Top Controls */}
      <div style={{ textAlign: 'center', marginBottom: 20 }}>
        <Space wrap>
          <Select
            value={authorFilter}
            onChange={setAuthorFilter}
            style={{ width: 200 }}
            options={authors.map((a) => ({ value: a, label: a }))}
          />
          <Button onClick={handleBulkDownload} loading={loading}>
            Download Selected (ZIP)
          </Button>
          <Button onClick={handleDeleteSelected} danger>
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
