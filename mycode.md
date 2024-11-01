import React, { useEffect, useState } from 'react';
import { callApi } from '../utils/Api';
import Footer from '../layouts/Footer';
import { Button } from 'reactstrap';
import PageTitle from '../components/PageTitle';
import { Link, useNavigate } from 'react-router-dom';
import { useDispatch, useSelector } from "react-redux";
import { fetchUserOrderSuccess, fetchUserOrderFailur } from '../actions/userOrderAction';

export default function MyOrder() {
  const dispatch = useDispatch();
  const userOrder = useSelector(state => state.userOrder);
  const [myOrder, setMyOrder] = useState(userOrder?.userOrder || []);
  const navigate = useNavigate();
  const userId = localStorage.getItem('userid');

  useEffect(() => {
    if (!userId) {
      navigate("/login");
    }
    if (userId) {
      callApi('post', 'cart/cartcrud/getUserWiseOrder', { userId: userId, limit: 100, page: 1 })
        .then((res) => {
          if (res.code === 200) {
            dispatch(fetchUserOrderSuccess(res.data));
          }
        }).catch((err) => {
          dispatch(fetchUserOrderFailur(err));
        });
    }
  }, [userId, navigate, dispatch]);

  useEffect(() => {
    setMyOrder(userOrder?.userOrder || []);
  }, [userOrder?.userOrder]);

  if (!userId) {
    return null;
  }

  // Group orders by date
  const ordersByDate = myOrder.reduce((acc, order) => {
    const orderDate = new Date(order.createdAt).toLocaleDateString('en-US', { year: 'numeric', month: 'long', day: 'numeric' });
    if (!acc[orderDate]) {
      acc[orderDate] = [];
    }
    acc[orderDate].push(order);
    return acc;
  }, {});

  return (
    <section className="h-100 gradient-custom">
      <PageTitle />
      <div className="container py-5 h-100">
        <div className="row d-flex justify-content-center align-items-center h-100">
          <div className="col-lg-10 col-xl-8">
            {Object.keys(ordersByDate).length !== 0 ?
              <div className="card" style={{ borderRadius: 10, border: 'none', boxShadow: '0 4px 8px rgba(0, 0, 0, 0.1)' }}>
                <div className="card-header px-4 py-5" style={{ backgroundColor: '#ff6f61', color: 'white' }}>
                  <h5 className="mb-0" style={{ fontWeight: 'bold' }}>Your Orders</h5>
                </div>
                <div className="card-body p-4">
                  {Object.entries(ordersByDate).map(([date, orders]) => (
                    <div key={date} className="mb-4">
                      <h6 style={{ fontWeight: 'bold', color: '#333' }}>{date}</h6>
                      {orders.map((order, orderIndex) => (
                        <div key={orderIndex} className="mb-3">
                          <div className="row mb-2">
                            <div className="col-md-6 col-6">
                              <p style={{ fontSize: '14px', color: '#555' }}>Order Id: <span className='order-page-order-date'>{order?.orderId}</span></p>
                            </div>
                            <div className="col-md-6 col-6 text-right">
                              <p style={{ fontSize: '14px', color: '#555' }}>Total amount paid: <span className='order-page-order-date'><i className='fa fa-inr'></i>{order?.totalAmount}</span></p>
                            </div>
                          </div>
                          {order.cart.map((car, indx) => (
                            <div key={indx} className="card shadow-0 border mb-4" style={{ borderRadius: '8px', border: '1px solid #e0e0e0' }}>
                              <div className="card-body">
                                <Link to={`/productdetails/rr/${car?.product?._id}`} style={{ textDecoration: 'none', color: 'inherit' }}>
                                  <div className="row">
                                    <div className="col-md-3">
                                      {car.productThumbnailImages?.[0].length !== 0 ? (
                                        <img
                                          src={car?.product?.productThumbnailImages?.[0]?.imgurl}
                                          className="img-fluid"
                                          alt="Product"
                                          style={{ borderRadius: '8px' }}
                                        />
                                      ) : (
                                        <img
                                          src="https://baconmockup.com/250/250/"  // Replace this with the actual URL of your default avatar image
                                          className="img-fluid"
                                          alt="Default Avatar"
                                          style={{ borderRadius: '8px' }}
                                        />
                                      )}
                                    </div>
                                    <div className="col-md-9 col-12">
                                      <div className="d-flex justify-content-between align-items-center">
                                        <p className="mb-0" style={{ fontWeight: 'bold', color: '#333' }}>{car?.product?.productName}</p>
                                        <p className="mb-0" style={{ fontSize: '14px', color: '#555' }}>Rs.{car?.product?.grossPrice}</p>
                                      </div>
                                      <div className="d-flex justify-content-between align-items-center mt-2">
                                        <p className="mb-0" style={{ fontSize: '14px', color: '#555' }}>Deliver by: <span className='order-page-order-date'>{car?.estimatedDeliveryDate}</span></p>
                                        <Link to={`/trackOrder/${car?._id}`} className="btn btn-primary btn-sm" style={{ backgroundColor: '#ff6f61', borderColor: '#ff6f61' }}>Track Order</Link>
                                      </div>
                                    </div>
                                  </div>
                                </Link>
                              </div>
                            </div>
                          ))}
                          <div className="row">
                            <div className="col-md-6 col-6">
                              <p className='order-page-order-date' style={{ fontSize: '14px', color: '#555' }}>Total Shipping charge</p>
                            </div>
                            <div className="col-md-6 col-6 text-right">
                              <span className='tax-text' style={{ fontSize: '14px', color: '#555' }}><i className='fa fa-inr'></i>{Math.round(order?.deliveryCharge)}</span>
                            </div>
                          </div>
                        </div>
                      ))}
                      <hr />
                    </div>
                  ))}
                </div>
                <br />
              </div> :
              <div className="card-header px-4 py-5" style={{ backgroundColor: '#f8f9fa', borderRadius: '10px', boxShadow: '0 4px 8px rgba(0, 0, 0, 0.1)' }}>
                <div className="text-center">
                  <img src="/assets/images/orderpage.png" alt="No Orders" className="img-fluid mb-4" style={{ maxWidth: '200px' }} />
                  <h5 className="text-muted mb-0">No Orders Yet</h5>
                  <p style={{ color: '#555' }}>You haven't placed any orders yet. Once you do, you can track them here! Start shopping now!</p>
                  <div style={{ marginTop: "40px" }}>
                    <Button
                      tag={Link}
                      to="/shop"
                      color="primary"
                      style={{ padding: "8px", borderRadius: '20px', backgroundColor: '#ff6f61', borderColor: '#ff6f61' }}
                    >
                      Back to Shopping
                    </Button>
                  </div>
                </div>
              </div>
            }
          </div>
        </div>
      </div>
      <Footer />
    </section>
  )
}
